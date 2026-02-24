# part 1 — foundations

the three core mocking tools in vitest, how they interact, and how to keep
them from leaking between tests. read this first if you're new to mocking.

---

## vi.fn() — standalone mock functions

> **when would I need this?** any time you need a fake function: a callback
> prop, an event handler, a function you pass to a hook, or any dependency
> you want to track calls on.

### creating and using a basic mock

```typescript
const handleClick = vi.fn();

render(<Button onClick={handleClick} label="Save" />);
await userEvent.setup().click(screen.getByRole('button', { name: /save/i }));

expect(handleClick).toHaveBeenCalledOnce();
expect(handleClick).toHaveBeenCalledWith(expect.any(Object)); // the click event
```

### controlling what a mock returns

```typescript
// fixed return value — every call returns the same thing
const getUser = vi.fn().mockReturnValue({ id: '1', name: 'Alice' });

// sequenced returns — each call returns the next value in line
const getNextId = vi.fn()
  .mockReturnValueOnce('id-1')
  .mockReturnValueOnce('id-2')
  .mockReturnValueOnce('id-3');

expect(getNextId()).toBe('id-1');
expect(getNextId()).toBe('id-2');
expect(getNextId()).toBe('id-3');
expect(getNextId()).toBeUndefined(); // exhausted — falls back to default
```

### mockImplementation — when return value depends on the input

```typescript
const formatCurrency = vi.fn().mockImplementation((amount: number) => {
  return `$${amount.toFixed(2)}`;
});

expect(formatCurrency(9.5)).toBe('$9.50');
expect(formatCurrency).toHaveBeenCalledWith(9.5);
```

### asserting call arguments in detail

```typescript
const onSubmit = vi.fn();

// ... render and interact ...

// exact match
expect(onSubmit).toHaveBeenCalledWith({ title: 'Buy milk', priority: 'high' });

// partial match — when you only care about some properties
expect(onSubmit).toHaveBeenCalledWith(
  expect.objectContaining({ title: 'Buy milk' })
);

// checking a specific call by position (1-indexed)
expect(onSubmit).toHaveBeenNthCalledWith(1, 'first call arg');
expect(onSubmit).toHaveBeenNthCalledWith(2, 'second call arg');

// inspecting raw call data directly
expect(onSubmit.mock.calls).toHaveLength(2);
expect(onSubmit.mock.calls[0][0]).toBe('first call arg');

// inspecting what the mock returned
expect(onSubmit.mock.results[0]).toEqual({ type: 'return', value: undefined });
```

---

## vi.spyOn() — observing without replacing

> **when would I need this?** when you want to verify a method was called
> but don't want to change what it does. also useful for temporarily
> suppressing console noise in tests.

### basic spy — verify a method was called

```typescript
const spy = vi.spyOn(console, 'warn');

render(<DeprecatedComponent />);

expect(spy).toHaveBeenCalledWith(
  expect.stringContaining('deprecated')
);

spy.mockRestore(); // restore original console.warn
```

### temporarily replacing an implementation

```typescript
// suppress console.error noise in a test that expects an error boundary
const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});

expect(() => {
  render(<BadComponent />);
}).toThrow();

consoleSpy.mockRestore();
```

### spying on a module's named export

```typescript
import * as analyticsModule from '../services/analytics';

const spy = vi.spyOn(analyticsModule, 'trackEvent');

render(<CheckoutButton />);
await userEvent.setup().click(screen.getByRole('button', { name: /checkout/i }));

expect(spy).toHaveBeenCalledWith('checkout_started', { source: 'button' });
```

### spying on a prototype method

```typescript
const spy = vi.spyOn(Storage.prototype, 'setItem');

render(<SettingsForm />);
await userEvent.setup().click(screen.getByRole('button', { name: /save/i }));

expect(spy).toHaveBeenCalledWith('user-settings', expect.any(String));

spy.mockRestore();
```

### vi.spyOn vs vi.fn — when to use which

| Scenario | Use |
|----------|-----|
| Callback prop to a component | `vi.fn()` — there's no original to spy on |
| Checking a module function was called | `vi.spyOn(module, 'fn')` — preserves original |
| Replacing a module function entirely | `vi.mock` with `vi.fn()` in the factory |
| Suppressing console noise | `vi.spyOn(console, 'error').mockImplementation(() => {})` |

---

## vi.mock() — replacing modules

> **when would I need this?** when you need to replace an entire module's
> exports — typically a custom hook, an API client, or a utility module — so
> your test doesn't depend on the real implementation.

### basic pattern

```typescript
import { useTodos } from '../hooks/useTodos';

// the factory function replaces the entire module.
// the import() form gives TypeScript path validation and IDE rename support.
// vi.mock('./path', ...) with a plain string also works.
vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

// vi.mocked() gives type-safe access to mock methods
const mockUseTodos = vi.mocked(useTodos);

mockUseTodos.mockReturnValue({ todos: [], loading: false, error: null });
```

### hoisting behavior — why vi.mock goes to the top

`vi.mock` calls are automatically hoisted to the top of the file by vitest's
transform, before all imports:

```typescript
// what you write:
import { useTodos } from '../hooks/useTodos';

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

// what vitest actually executes:
vi.mock(import('../hooks/useTodos'), () => ({  // ← runs first
  useTodos: vi.fn(),
}));

import { useTodos } from '../hooks/useTodos';   // ← useTodos is already the mock
```

the factory function **cannot reference variables** declared in the file
because those variables don't exist yet when the factory runs:

```typescript
// BAD: returnValue is not defined when the factory runs
const returnValue = { todos: [], loading: false };

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(() => returnValue), // ReferenceError!
}));
```

### vi.mock is file-scoped

`vi.mock` applies to the entire test file and cannot be undone per-test.
vitest warns if `vi.mock` or `vi.hoisted` appear inside a `describe` block,
`if` statement, or function — they are compiler hints and always escape
their scope. use separate test files if some tests need the real module and
others need it mocked.

---

## vi.hoisted — declaring variables accessible in the mock factory

> **when would I need this?** when you need to reference a mock inside the
> `vi.mock` factory function, or when you want to set up a mock reference
> without importing the real module.

```typescript
// vi.hoisted runs at the top of the file, before imports and vi.mock factories
const { mockUseTodos } = vi.hoisted(() => ({
  mockUseTodos: vi.fn(),
}));

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: mockUseTodos, // ← this works because vi.hoisted ran first
}));

describe('TodoList', () => {
  it('renders todos', () => {
    mockUseTodos.mockReturnValue({
      todos: [{ id: '1', title: 'Test', completed: false }],
      loading: false,
      error: null,
    });

    render(<TodoList />);
    expect(screen.getByText('Test')).toBeInTheDocument();
  });
});
```

### vi.mocked vs vi.hoisted — when to use each

| Approach | Use when |
|----------|----------|
| `vi.mocked(importedFn)` | You already import the function and need type-safe access to mock methods |
| `vi.hoisted` | You need the mock reference *inside* the factory, or want to avoid importing the real module |

---

## partial module mocking with importOriginal

> **when would I need this?** when a module has many exports but you only
> want to mock one or two. keeps the rest working as real code.

```typescript
vi.mock(import('../utils/dateHelpers'), async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    formatDate: vi.fn(() => '01/01/2025'), // mock just this one
    // getNow, parseDate, etc. remain the real implementations
  };
});
```

---

## mocking default exports

> **when would I need this?** when the module uses `export default` instead
> of named exports.

```typescript
vi.mock(import('../hooks/useSomething'), () => ({
  default: vi.fn(), // ← the key must be "default"
}));

import useSomething from '../hooks/useSomething';
const mockUseSomething = vi.mocked(useSomething);
```

---

## mocking class exports

> **when would I need this?** when a module exports a class (like an API
> client) and you need to control what its methods return.

### basic class mock

```typescript
// services/ApiClient.ts
export class ApiClient {
  async get(url: string) { /* ... */ }
  async post(url: string, body: unknown) { /* ... */ }
}
```

```typescript
vi.mock(import('../services/ApiClient'), () => {
  const MockApiClient = vi.fn().mockImplementation(() => ({
    get: vi.fn().mockResolvedValue({ data: [] }),
    post: vi.fn().mockResolvedValue({ data: { id: '1' } }),
  }));

  return { ApiClient: MockApiClient };
});
```

### per-test control with vi.hoisted

```typescript
const { mockGet, mockPost } = vi.hoisted(() => ({
  mockGet: vi.fn(),
  mockPost: vi.fn(),
}));

vi.mock(import('../services/ApiClient'), () => ({
  ApiClient: vi.fn().mockImplementation(() => ({
    get: mockGet,
    post: mockPost,
  })),
}));

it('calls GET on mount', async () => {
  mockGet.mockResolvedValueOnce({ data: [{ id: '1', name: 'Test' }] });

  render(<UserList />);

  expect(await screen.findByText('Test')).toBeInTheDocument();
  expect(mockGet).toHaveBeenCalledWith('/users');
});
```

---

## mock lifecycle: mockClear vs mockReset vs mockRestore

> **when would I need this?** when you're confused about why a mock still
> has state from a previous test, or when choosing how to clean up mocks
> between tests.

| Method | Clears call history | Removes mock implementation | Restores original |
|--------|--------------------|-----------------------------|-------------------|
| `mockClear()` | yes | no | no |
| `mockReset()` | yes | yes (returns `undefined`) | no |
| `mockRestore()` | yes | yes | yes (only for `vi.spyOn`) |

### practical examples

```typescript
const mockFn = vi.fn(() => 'mocked value');

mockFn('arg1');
console.log(mockFn.mock.calls); // [['arg1']]
console.log(mockFn());          // 'mocked value'

// mockClear: resets history, keeps implementation
mockFn.mockClear();
console.log(mockFn.mock.calls); // []
console.log(mockFn());          // 'mocked value' — still works

// mockReset: resets history AND implementation
mockFn.mockReset();
console.log(mockFn.mock.calls); // []
console.log(mockFn());          // undefined — implementation gone

// mockRestore: only meaningful for spies
const spy = vi.spyOn(console, 'log').mockImplementation(() => {});
spy.mockRestore(); // console.log works normally again
```

### recommended reset strategy

in vitest 3 and earlier, `restoreAllMocks: true` in the config was the standard
recommendation. **vitest 4 changed this behavior:** `vi.restoreAllMocks()` now
only restores spies created with `vi.spyOn` — it no longer resets mocks created
with `vi.fn()` or automocked modules.

for vitest 4, use `mockReset` as the default:

```typescript
// vitest.config.ts — vitest 4
export default defineConfig({
  test: {
    mockReset: true, // resets all vi.fn() mocks between tests
  },
});
```

or per-file:

```typescript
beforeEach(() => {
  vi.resetAllMocks();
});
```

if you also use `vi.spyOn`, combine both:

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    mockReset: true,
    restoreMocks: true,
  },
});
```

| config option | vitest 4 behavior |
|---------------|-------------------|
| `mockReset: true` | calls `mockReset()` on all `vi.fn()` mocks after each test |
| `restoreMocks: true` | calls `mockRestore()` on `vi.spyOn` spies only (no longer affects `vi.fn`) |
| `clearMocks: true` | clears call history only, keeps implementations |

none of these undo `vi.mock()`. module-level mocks persist for the entire file.
to unmock a module, use `vi.unmock('./path')`.
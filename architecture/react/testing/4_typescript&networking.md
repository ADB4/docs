# part 4 — typescript and troubleshooting

type-safe mocking patterns, the most common mistakes (with fixes), and a
quick-reference table for looking up the right tool.

---

## type-safe mocking patterns

### vi.mocked — deep type inference

> **when would I need this?** whenever you mock a module with `vi.mock` and
> want TypeScript to know the return type of `mockReturnValue`,
> `mockResolvedValue`, etc.

```typescript
import { fetchTodos } from '../api/todos';

vi.mock(import('../api/todos'));

// vi.mocked wraps the function with mock type information
const mockFetchTodos = vi.mocked(fetchTodos);

// TypeScript now knows mockFetchTodos has:
// - .mockReturnValue() expects the correct return type
// - .mockResolvedValue() expects the correct resolved type
// - .mock.calls has the correct argument types

mockFetchTodos.mockResolvedValue([
  { id: '1', title: 'Test', completed: false },
  // TypeScript error if you add unknown properties or miss required ones
]);
```

### typing the mock factory with satisfies

> **when would I need this?** when you build a factory helper for a mocked
> hook and want TypeScript to catch mismatches between your factory output
> and the real hook's return type.

```typescript
import type { UseTodosReturn } from '../hooks/useTodos';

const createMockReturn = (
  overrides: Partial<UseTodosReturn> = {}
): UseTodosReturn => ({
  todos: [],
  loading: false,
  error: null,
  addTodo: vi.fn(),
  toggleTodo: vi.fn(),
  deleteTodo: vi.fn(),
  ...overrides,
}) satisfies UseTodosReturn;
// satisfies ensures the object structure matches without widening the type
```

### MockInstance type for storing mock references

> **when would I need this?** when you store a mock in a `let` variable
> (e.g. in `beforeEach`) and need to tell TypeScript what it is.

```typescript
import type { MockInstance } from 'vitest';

let fetchSpy: MockInstance<typeof fetch>;

beforeEach(() => {
  fetchSpy = vi.fn().mockResolvedValue({
    ok: true,
    json: () => Promise.resolve([]),
  } as Response);
  vi.stubGlobal('fetch', fetchSpy);
});
```

---

## common mocking mistakes

### mistake: mock return value shape doesn't match the real hook

**symptom:** `Cannot read properties of undefined` when rendering.

```typescript
// BAD: missing properties the component destructures
vi.mocked(useTodos).mockReturnValue({
  todos: [],
  // forgot loading, error, addTodo, etc.
});

// GOOD: always return the complete shape
vi.mocked(useTodos).mockReturnValue({
  todos: [],
  loading: false,
  error: null,
  addTodo: vi.fn(),
  toggleTodo: vi.fn(),
  deleteTodo: vi.fn(),
});
```

TypeScript catches this if your hook has a well-typed return type —
`vi.mocked(useTodos).mockReturnValue(...)` will show type errors for
missing properties.

**fix:** use a mock factory helper (see part 2) so you never have to
remember the full shape.

### mistake: not resetting mocks between tests

**symptom:** a test passes in isolation but fails when run with other tests,
or tests behave differently depending on execution order.

```typescript
// BAD: test 2 inherits mock state from test 1
it('test 1', () => {
  vi.mocked(useTodos).mockReturnValue({ todos: [...], loading: false, ... });
  render(<TodoList />);
});

it('test 2', () => {
  // useTodos still returns the value from test 1!
  render(<TodoList />);
});

// GOOD: reset in beforeEach (or use mockReset: true in config)
beforeEach(() => {
  vi.mocked(useTodos).mockReturnValue(createMockUseTodosReturn());
});
```

**fix:** set `mockReset: true` in your vitest config (see part 1, mock
lifecycle section).

### mistake: mocking too deep

**symptom:** tests become brittle, break when internal implementation
changes, or require mocking React internals.

```typescript
// BAD: mocking React internals
vi.mock(import('react'), async (importOriginal) => ({
  ...(await importOriginal()),
  useState: vi.fn(), // never do this
}));

// GOOD: mock at the boundary of your code
// mock your custom hooks, your API layer, or external services
// never mock React, ReactDOM, or testing-library internals
```

**rule of thumb:** if you're mocking something you didn't write *and* it's
not a network call or environment API, you're probably mocking too deep.

### mistake: testing the mock instead of the component

**symptom:** the test passes but doesn't actually verify any user-visible
behavior.

```typescript
// BAD: asserting on mock state without user interaction
it('calls addTodo', () => {
  const mockAddTodo = vi.fn();
  vi.mocked(useTodos).mockReturnValue({
    ...createMockUseTodosReturn(),
    addTodo: mockAddTodo,
  });

  render(<TodoList />);

  expect(mockAddTodo).not.toHaveBeenCalled(); // this tells you nothing useful
});

// GOOD: simulate the user action that triggers the hook call
it('calls addTodo when user submits the form', async () => {
  const mockAddTodo = vi.fn();
  const user = userEvent.setup();

  vi.mocked(useTodos).mockReturnValue({
    ...createMockUseTodosReturn(),
    addTodo: mockAddTodo,
  });

  render(<TodoList />);

  await user.type(screen.getByRole('textbox', { name: /task/i }), 'New todo');
  await user.click(screen.getByRole('button', { name: /add/i }));

  expect(mockAddTodo).toHaveBeenCalledWith('New todo');
});
```

**rule of thumb:** if your test doesn't call `userEvent` or interact with
the component somehow, ask yourself what behavior you're actually verifying.

### mistake: using jest.requireActual in vitest

**symptom:** `TypeError: jest is not defined` or `jest.requireActual is not
a function`.

```typescript
// BAD: jest API — does not exist in vitest
vi.mock(import('../utils'), async () => ({
  ...jest.requireActual('../utils'), // TypeError
  formatDate: vi.fn(),
}));

// GOOD: use importOriginal (vitest's equivalent)
vi.mock(import('../utils'), async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    formatDate: vi.fn(),
  };
});
```

### mistake: putting vi.mock inside describe or it blocks

**symptom:** vitest warns that `vi.mock` was found inside a function body.
the mock still applies to the entire file, which may cause surprising
behavior.

```typescript
// BAD: vi.mock inside describe — it escapes anyway
describe('with mocked hook', () => {
  vi.mock(import('../hooks/useTodos'), () => ({
    useTodos: vi.fn(),
  }));

  it('test 1', () => { /* useTodos is mocked */ });
});

describe('with real hook', () => {
  it('test 2', () => { /* useTodos is STILL mocked — vi.mock is file-scoped */ });
});

// GOOD: put vi.mock at the top level and use separate test files
// when some tests need the real module
```

### mistake: forgetting advanceTimers with user-event and fake timers

**symptom:** `user.type()` or `user.click()` hangs or times out when
`vi.useFakeTimers()` is active.

```typescript
// BAD: user-event's internal delays freeze because timers are fake
vi.useFakeTimers();
const user = userEvent.setup();
await user.type(input, 'hello'); // hangs forever

// GOOD: tell user-event how to advance fake timers
vi.useFakeTimers();
const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
await user.type(input, 'hello'); // works
```

---

## quick reference: which mocking tool to reach for

| I need to… | Use | File |
|---|---|---|
| Create a mock callback for a component prop | `vi.fn()` | 01 |
| Control what a mock function returns | `.mockReturnValue()` / `.mockReturnValueOnce()` | 01 |
| Make a mock function return a resolved promise | `.mockResolvedValue()` / `.mockResolvedValueOnce()` | 03 |
| Make a mock function return a rejected promise | `.mockRejectedValue()` / `.mockRejectedValueOnce()` | 03 |
| Check that an existing method was called without replacing it | `vi.spyOn(object, 'method')` | 01 |
| Replace an entire module's exports | `vi.mock(import('./mod'), () => ({ ... }))` | 01 |
| Replace one export but keep the rest real | `vi.mock` with `importOriginal` spread | 01 |
| Reference a variable inside a vi.mock factory | `vi.hoisted(() => ({ ref: vi.fn() }))` | 01 |
| Get type-safe access to a mocked import | `vi.mocked(importedFn)` | 04 |
| Mock a global (fetch, location) | `vi.stubGlobal('name', mock)` | 03 |
| Control time (setTimeout, Date) | `vi.useFakeTimers()` + `vi.advanceTimersByTimeAsync()` | 03 |
| Intercept network requests realistically | MSW `setupServer` + handlers | 03 |
| Mock a custom hook in a component test | `vi.mock` the hook module, `vi.mocked` to control it | 02 |
| Mock a heavy child component | `vi.mock` returning a simple JSX stand-in | 02 |
| Wrap a component in providers for testing | Custom render function with `wrapper` | 02 |
| Mock React Router hooks | `vi.mock('react-router-dom')` or `<MemoryRouter>` | 02 |
| Mock a class export (API client, service) | `vi.mock` with `vi.fn().mockImplementation(...)` | 01 |
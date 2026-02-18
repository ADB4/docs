## 6. mocking custom hooks in component tests

### basic pattern: mocking a hook module

```typescript
// components/TodoList.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoList } from './TodoList';
import { useTodos } from '../hooks/useTodos';

// vi.mock is hoisted to the top of the file before imports.
// the factory function replaces the entire module.
// the import() form gives TypeScript path validation and IDE rename support.
// vi.mock('./path', ...) with a plain string also works.
vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

describe('TodoList', () => {
  // vi.mocked() gives type-safe access to mock methods
  const mockUseTodos = vi.mocked(useTodos);

  beforeEach(() => {
    // reset to a sensible default before each test
    mockUseTodos.mockReturnValue({
      todos: [],
      loading: false,
      error: null,
      addTodo: vi.fn(),
      toggleTodo: vi.fn(),
      deleteTodo: vi.fn(),
    });
  });

  it('renders a list of todos', () => {
    mockUseTodos.mockReturnValue({
      todos: [
        { id: '1', title: 'Buy milk', completed: false },
        { id: '2', title: 'Walk dog', completed: true },
      ],
      loading: false,
      error: null,
      addTodo: vi.fn(),
      toggleTodo: vi.fn(),
      deleteTodo: vi.fn(),
    });

    render(<TodoList />);

    expect(screen.getByText('Buy milk')).toBeInTheDocument();
    expect(screen.getByText('Walk dog')).toBeInTheDocument();
  });

  it('shows loading spinner when loading', () => {
    mockUseTodos.mockReturnValue({
      todos: [],
      loading: true,
      error: null,
      addTodo: vi.fn(),
      toggleTodo: vi.fn(),
      deleteTodo: vi.fn(),
    });

    render(<TodoList />);

    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });

  it('shows error message on failure', () => {
    mockUseTodos.mockReturnValue({
      todos: [],
      loading: false,
      error: 'Failed to fetch todos',
      addTodo: vi.fn(),
      toggleTodo: vi.fn(),
      deleteTodo: vi.fn(),
    });

    render(<TodoList />);

    expect(screen.getByText(/failed to fetch todos/i)).toBeInTheDocument();
  });

  it('calls deleteTodo when delete button is clicked', async () => {
    const mockDeleteTodo = vi.fn();
    const user = userEvent.setup();

    mockUseTodos.mockReturnValue({
      todos: [{ id: '1', title: 'Test todo', completed: false }],
      loading: false,
      error: null,
      addTodo: vi.fn(),
      toggleTodo: vi.fn(),
      deleteTodo: mockDeleteTodo,
    });

    render(<TodoList />);

    await user.click(screen.getByRole('button', { name: /delete/i }));

    expect(mockDeleteTodo).toHaveBeenCalledWith('1');
  });
});
```

### reducing boilerplate with a mock factory helper

```typescript
// test-helpers/mockUseTodos.ts
import { vi } from 'vitest';

interface MockUseTodosReturn {
  todos: Array<{ id: string; title: string; completed: boolean }>;
  loading: boolean;
  error: string | null;
  addTodo: ReturnType<typeof vi.fn>;
  toggleTodo: ReturnType<typeof vi.fn>;
  deleteTodo: ReturnType<typeof vi.fn>;
}

const defaults: MockUseTodosReturn = {
  todos: [],
  loading: false,
  error: null,
  addTodo: vi.fn(),
  toggleTodo: vi.fn(),
  deleteTodo: vi.fn(),
};

// pass only the fields you care about per test
export const createMockUseTodosReturn = (
  overrides: Partial<MockUseTodosReturn> = {}
): MockUseTodosReturn => ({
  ...defaults,
  // recreate fresh vi.fn() instances to avoid cross-test leakage
  addTodo: vi.fn(),
  toggleTodo: vi.fn(),
  deleteTodo: vi.fn(),
  ...overrides,
});
```

```typescript
// components/TodoList.test.tsx — using the helper
import { useTodos } from '../hooks/useTodos';
import { createMockUseTodosReturn } from '../test-helpers/mockUseTodos';

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

const mockUseTodos = vi.mocked(useTodos);

describe('TodoList', () => {
  beforeEach(() => {
    mockUseTodos.mockReturnValue(createMockUseTodosReturn());
  });

  it('renders todos', () => {
    mockUseTodos.mockReturnValue(
      createMockUseTodosReturn({
        todos: [{ id: '1', title: 'Buy milk', completed: false }],
      })
    );

    render(<TodoList />);
    expect(screen.getByText('Buy milk')).toBeInTheDocument();
  });

  it('shows loading state', () => {
    mockUseTodos.mockReturnValue(
      createMockUseTodosReturn({ loading: true })
    );

    render(<TodoList />);
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });

  it('calls addTodo when user submits the form', async () => {
    const mockAddTodo = vi.fn();
    const user = userEvent.setup();

    mockUseTodos.mockReturnValue(
      createMockUseTodosReturn({ addTodo: mockAddTodo })
    );

    render(<TodoList />);

    await user.type(screen.getByRole('textbox', { name: /task/i }), 'New todo');
    await user.click(screen.getByRole('button', { name: /add/i }));

    expect(mockAddTodo).toHaveBeenCalledWith('New todo');
  });
});
```

---

## 7. vi.mock mechanics and advanced patterns

### vi.mock hoisting behavior

`vi.mock` calls are automatically hoisted to the top of the file by vitest's
transform, before all imports:

```typescript
// what you write:
import { useTodos } from '../hooks/useTodos';

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

// what vitest executes:
vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

import { useTodos } from '../hooks/useTodos';
// useTodos is already the mock by the time the import resolves
```

the factory function cannot reference variables declared in the file because
those variables don't exist yet when the factory runs:

```typescript
// BAD: returnValue is not defined when the factory runs
const returnValue = { todos: [], loading: false };

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(() => returnValue), // ReferenceError!
}));
```

### vi.hoisted — declaring variables accessible in the mock factory

```typescript
// vi.hoisted runs at the top of the file, before imports and vi.mock factories
const { mockUseTodos } = vi.hoisted(() => ({
  mockUseTodos: vi.fn(),
}));

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: mockUseTodos,
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

when to use each approach:

| Approach | Use when |
|----------|----------|
| `vi.mocked(importedFn)` | You already import the function and need type-safe access to mock methods |
| `vi.hoisted` | You need the mock reference *inside* the factory, or want to avoid importing the real module |

### partial module mocking with importOriginal

mock only specific exports while keeping everything else real:

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

### mocking a hook that uses other hooks internally

**strategy 1: mock the custom hook entirely (preferred for component tests)**

```typescript
// if TodoDetail uses useGetTodo which internally calls useQuery + useParams,
// mock the custom hook — don't go deeper
vi.mock(import('../hooks/useGetTodo'), () => ({
  useGetTodo: vi.fn(),
}));

vi.mocked(useGetTodo).mockReturnValue({
  data: { id: '1', title: 'Test', completed: false },
  isLoading: false,
  error: null,
});
```

**strategy 2: mock the underlying dependencies (for hook integration tests)**

```typescript
// when testing the hook itself, mock what IT depends on
vi.mock(import('react-router-dom'), async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    useParams: vi.fn(() => ({ id: '1' })),
    useNavigate: vi.fn(() => vi.fn()),
  };
});

// then test the hook with renderHook
const { result } = renderHook(() => useGetTodo(), {
  wrapper: createQueryWrapper(),
});
```

### mocking default exports

```typescript
// for: export default useSomething
vi.mock(import('../hooks/useSomething'), () => ({
  default: vi.fn(),
}));

import useSomething from '../hooks/useSomething';
const mockUseSomething = vi.mocked(useSomething);
```

### mocking a hook that returns a ref

```typescript
vi.mocked(useMyRef).mockReturnValue({
  current: document.createElement('div'),
});

// for a callback ref pattern:
const mockRef = vi.fn();
vi.mocked(useCallbackRef).mockReturnValue(mockRef);
```

### vi.mock is file-scoped

`vi.mock` applies to the entire test file and cannot be undone per-test.
recent versions of vitest warn if `vi.mock` or `vi.hoisted` appear inside a
function, `describe` block, or `if` statement — they are compiler hints and
always escape their scope. use separate test files if some tests need the real
hook and others need it mocked.

```typescript
// vi.mock applies here — ALL tests in this file get the mock
vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

it('test with real hook', () => {
  // useTodos is STILL mocked — you cannot undo vi.mock per-test
});
```

---

## 8. mock lifecycle: mockClear vs mockReset vs mockRestore

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

---

## 9. mocking third-party hooks

### react router

```typescript
vi.mock(import('react-router-dom'), async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    useNavigate: vi.fn(),
    useParams: vi.fn(),
    useSearchParams: vi.fn(),
  };
});

import { useNavigate, useParams } from 'react-router-dom';

describe('TodoDetail', () => {
  it('navigates back on close', async () => {
    const mockNavigate = vi.fn();
    vi.mocked(useNavigate).mockReturnValue(mockNavigate);
    vi.mocked(useParams).mockReturnValue({ id: '1' });

    const user = userEvent.setup();
    render(<TodoDetail />);

    await user.click(screen.getByRole('button', { name: /close/i }));

    expect(mockNavigate).toHaveBeenCalledWith('/todos');
  });
});
```

alternative for integration-style tests — wrap in `<MemoryRouter>` instead of
mocking:

```typescript
render(
  <MemoryRouter initialEntries={['/todos/1']}>
    <Routes>
      <Route path="/todos/:id" element={<TodoDetail />} />
    </Routes>
  </MemoryRouter>
);
```

### useMediaQuery / window.matchMedia

```typescript
beforeEach(() => {
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: vi.fn().mockImplementation((query: string) => ({
      matches: query === '(min-width: 600px)',
      media: query,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    })),
  });
});
```

---

## 10. common mocking mistakes

### mistake: mock return value shape doesn't match the real hook

```typescript
// BAD: missing properties the component destructures
vi.mocked(useTodos).mockReturnValue({
  todos: [],
  // forgot loading, error, addTodo, etc.
  // component will crash: Cannot read properties of undefined
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

typescript catches this if your hook has a well-typed return type —
`vi.mocked(useTodos).mockReturnValue(...)` will show type errors for
missing properties.

### mistake: not resetting mocks between tests

```typescript
// BAD: test 2 inherits mock state from test 1
it('test 1', () => {
  vi.mocked(useTodos).mockReturnValue({ todos: [...], loading: false, ... });
  render(<TodoList />);
});

it('test 2', () => {
  // useTodos still returns the value from test 1
  render(<TodoList />);
});

// GOOD: reset in beforeEach (or use mockReset: true in config)
beforeEach(() => {
  vi.mocked(useTodos).mockReturnValue(createMockUseTodosReturn());
});
```

### mistake: mocking too deep

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

### mistake: testing the mock instead of the component

```typescript
// BAD: asserting on mock state without user interaction
it('calls addTodo', () => {
  const mockAddTodo = vi.fn();
  vi.mocked(useTodos).mockReturnValue({
    ...createMockUseTodosReturn(),
    addTodo: mockAddTodo,
  });

  render(<TodoList />);

  expect(mockAddTodo).not.toHaveBeenCalled(); // this tells you nothing
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

### mistake: using jest.requireActual in vitest

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
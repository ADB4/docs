# part 2 — component testing patterns

patterns for isolating components in tests by mocking their dependencies:
custom hooks, child components, context providers, and third-party libraries.

---

## mocking custom hooks in component tests

> **when would I need this?** when a component uses a custom hook for data
> fetching, state management, or side effects, and you want to test the
> component's rendering and interactions in isolation from the hook's
> implementation.

### basic pattern

```typescript
// components/TodoList.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoList } from './TodoList';
import { useTodos } from '../hooks/useTodos';

vi.mock(import('../hooks/useTodos'), () => ({
  useTodos: vi.fn(),
}));

describe('TodoList', () => {
  const mockUseTodos = vi.mocked(useTodos);

  beforeEach(() => {
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

> **when would I need this?** when your mocked hook has many properties and
> you're tired of repeating the full return shape in every test. the factory
> helper lets each test specify only the fields it cares about.

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

### mocking a hook that uses other hooks internally

> **when would I need this?** when you need to decide at what level to
> mock. the rule of thumb: mock your custom hooks for component tests, mock
> the underlying dependencies only when testing the hook itself.

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

### mocking a hook that returns a ref

```typescript
vi.mocked(useMyRef).mockReturnValue({
  current: document.createElement('div'),
});

// for a callback ref pattern:
const mockRef = vi.fn();
vi.mocked(useCallbackRef).mockReturnValue(mockRef);
```

---

## mocking child components

> **when would I need this?** when a child component is expensive to render
> (charts, maps, rich text editors), has side effects (network calls,
> animations), or makes the test slow/flaky. mock the child so you can
> focus on the parent's behavior.

### basic child component mock

```typescript
vi.mock(import('../components/ExpensiveChart'), () => ({
  ExpensiveChart: ({ data }: { data: unknown[] }) => (
    <div data-testid="chart-mock">Chart: {data.length} points</div>
  ),
}));

it('renders the dashboard with chart data', () => {
  render(<Dashboard />);
  expect(screen.getByTestId('chart-mock')).toHaveTextContent('Chart: 5 points');
});
```

### verifying props passed to a mocked child

```typescript
import { ExpensiveChart } from '../components/ExpensiveChart';

vi.mock(import('../components/ExpensiveChart'), () => ({
  ExpensiveChart: vi.fn(() => <div data-testid="chart-mock" />),
}));

it('passes filtered data to the chart', () => {
  render(<Dashboard filter="active" />);

  expect(vi.mocked(ExpensiveChart)).toHaveBeenCalledWith(
    expect.objectContaining({
      data: expect.arrayContaining([
        expect.objectContaining({ status: 'active' }),
      ]),
    }),
    expect.anything() // ref argument — always present in React component calls
  );
});
```

---

## mocking context providers

> **when would I need this?** when your component requires one or more
> context providers (theme, query client, auth, etc.) to render. instead of
> mocking the provider itself, wrap your component in real providers with
> controlled values.

### custom render wrapper (recommended approach)

```typescript
// test-helpers/renderWithProviders.tsx
import { render, type RenderOptions } from '@testing-library/react';
import { ThemeProvider, createTheme } from '@mui/material';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

interface ExtendedRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  theme?: ReturnType<typeof createTheme>;
  queryClient?: QueryClient;
}

export function renderWithProviders(
  ui: React.ReactElement,
  {
    theme = createTheme(),
    queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    }),
    ...renderOptions
  }: ExtendedRenderOptions = {}
) {
  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        <ThemeProvider theme={theme}>
          {children}
        </ThemeProvider>
      </QueryClientProvider>
    );
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
    queryClient,
  };
}
```

```typescript
// usage
it('renders with theme and query support', async () => {
  renderWithProviders(<TodoList />);
  expect(await screen.findByText('My Todos')).toBeInTheDocument();
});
```

### alternative: mock the useContext-based hook

if you only need to control a single context value and don't want to set up
providers, mock the hook that reads from the context instead (see the
"mocking custom hooks" section above).

### TanStack Query — always disable retries in tests

```typescript
// without this, failed queries retry 3 times with exponential backoff,
// making tests slow and flaky
const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});
```

---

## mocking third-party hooks

### react router

> **when would I need this?** when your component uses `useNavigate`,
> `useParams`, `useSearchParams`, or other React Router hooks.

**approach 1: mock the hooks (unit-test style)**

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

**approach 2: wrap in MemoryRouter (integration-test style)**

```typescript
// no mocking needed — uses real router behavior
render(
  <MemoryRouter initialEntries={['/todos/1']}>
    <Routes>
      <Route path="/todos/:id" element={<TodoDetail />} />
    </Routes>
  </MemoryRouter>
);
```

use MemoryRouter when you want to test that route params, navigation, and
URL changes work together. use mocks when you want to isolate the component
from routing entirely.

### useMediaQuery / window.matchMedia

> **when would I need this?** when your component or a library hook
> (e.g. MUI's `useMediaQuery`) depends on `window.matchMedia`, which
> jsdom doesn't implement.

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
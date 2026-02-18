# react testing library + vitest cheat sheet

## for react/typescript/mui applications

---

## table of contents

1. [core testing patterns](#1-core-testing-patterns)
2. [testing forms (text, date, autocomplete)](#2-testing-forms-text-date-autocomplete)
3. [testing api calls](#3-testing-api-calls)
4. [testing custom hooks](#4-testing-custom-hooks)
5. [common pitfalls and solutions](#5-common-pitfalls-and-solutions)

---

## 1. core testing patterns

### basic test structure

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from './test-utils';
import userEvent from '@testing-library/user-event';
import { TodoList } from './TodoList';

describe('TodoList', () => {
  it('renders the todo list title', () => {
    render(<TodoList />);
    expect(screen.getByRole('heading', { name: /my todos/i })).toBeInTheDocument();
  });
});
```

### query priority (use in this order)

| Priority | Query Type | Use Case |
|----------|------------|----------|
| 1 | `getByRole` | Accessible elements (buttons, inputs, headings) |
| 2 | `getByLabelText` | Form fields with labels |
| 3 | `getByPlaceholderText` | Inputs with placeholder text |
| 4 | `getByText` | Non-interactive text content |
| 5 | `getByDisplayValue` | Current value of form elements |
| 6 | `getByAltText` | Images, areas, custom elements with alt text |
| 7 | `getByTitle` | Elements with a title attribute |
| 8 | `getByTestId` | Last resort when no semantic query works |

### query variants

```typescript
// synchronous - throws if not found
screen.getByRole('button', { name: /submit/i });

// synchronous - returns null if not found
screen.queryByRole('button', { name: /submit/i });

// asynchronous - waits for element to appear
await screen.findByRole('button', { name: /submit/i });

// multiple elements
screen.getAllByRole('listitem');
screen.queryAllByRole('listitem');
await screen.findAllByRole('listitem');
```

### user event setup

```typescript
import userEvent from '@testing-library/user-event';

describe('TodoForm', () => {
  it('allows typing in the input', async () => {
    const user = userEvent.setup();
    render(<TodoForm />);

    const input = screen.getByRole('textbox', { name: /task/i });
    await user.type(input, 'Buy groceries');

    expect(input).toHaveValue('Buy groceries');
  });
});
```

### common assertions

```typescript
// presence
expect(element).toBeInTheDocument();
expect(element).not.toBeInTheDocument();

// visibility
expect(element).toBeVisible();
expect(element).not.toBeVisible();

// content
expect(element).toHaveTextContent('Expected text');
expect(element).toHaveValue('input value');

// attributes
expect(element).toHaveAttribute('disabled');
expect(element).toHaveClass('active');
expect(element).toBeDisabled();
expect(element).toBeEnabled();

// form validation
expect(element).toBeValid();
expect(element).toBeInvalid();
expect(element).toBeRequired();
```

### testing click handlers and callbacks

```typescript
it('calls onDelete when delete button is clicked', async () => {
  const user = userEvent.setup();
  const handleDelete = vi.fn();

  render(<TodoItem todo={mockTodo} onDelete={handleDelete} />);

  await user.click(screen.getByRole('button', { name: /delete/i }));

  expect(handleDelete).toHaveBeenCalledTimes(1);
  expect(handleDelete).toHaveBeenCalledWith(mockTodo.id);
});
```

### testing conditional rendering

```typescript
it('shows empty state when no todos exist', () => {
  render(<TodoList todos={[]} />);

  expect(screen.getByText(/no todos yet/i)).toBeInTheDocument();
  expect(screen.queryByRole('list')).not.toBeInTheDocument();
});

it('renders todo items when todos exist', () => {
  const todos = [
    { id: '1', title: 'Buy milk', completed: false },
    { id: '2', title: 'Walk dog', completed: true },
  ];

  render(<TodoList todos={todos} />);

  expect(screen.queryByText(/no todos yet/i)).not.toBeInTheDocument();
  expect(screen.getAllByRole('listitem')).toHaveLength(2);
});
```

---

## 2. testing forms (text, date, autocomplete)

### example todo form component

```typescript
// TodoForm.tsx
interface TodoFormProps {
  onSubmit: (todo: {
    title: string;
    dueDate: Date | null;
    assignee: string | null;
  }) => void;
}

export const TodoForm = ({ onSubmit }: TodoFormProps) => {
  const [title, setTitle] = useState('');
  const [dueDate, setDueDate] = useState<Date | null>(null);
  const [assignee, setAssignee] = useState<string | null>(null);

  const teamMembers = ['Alice', 'Bob', 'Charlie', 'Diana'];

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ title, dueDate, assignee });
  };

  return (
    <form onSubmit={handleSubmit}>
      <TextField
        label="Task Title"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        required
      />
      <DatePicker
        label="Due Date"
        value={dueDate}
        onChange={setDueDate}
      />
      <Autocomplete
        options={teamMembers}
        value={assignee}
        onChange={(_, value) => setAssignee(value)}
        renderInput={(params) => <TextField {...params} label="Assignee" />}
      />
      <Button type="submit">Add Todo</Button>
    </form>
  );
};
```

### testing text input

```typescript
describe('TodoForm - Text Input', () => {
  it('submits form with entered title', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    const titleInput = screen.getByRole('textbox', { name: /task title/i });
    await user.type(titleInput, 'New todo item');

    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(handleSubmit).toHaveBeenCalledWith(
      expect.objectContaining({ title: 'New todo item' })
    );
  });

  it('validates required title field', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    // try to submit without title
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    // form should not submit
    expect(handleSubmit).not.toHaveBeenCalled();
  });

  it('clears and replaces input text', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const titleInput = screen.getByRole('textbox', { name: /task title/i });

    await user.type(titleInput, 'Original text');
    expect(titleInput).toHaveValue('Original text');

    await user.clear(titleInput);
    expect(titleInput).toHaveValue('');

    await user.type(titleInput, 'New text');
    expect(titleInput).toHaveValue('New text');
  });
});
```

### testing MUI datepicker

```typescript
describe('TodoForm - Date Picker', () => {
  it('allows entering a due date via keyboard', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    // type date directly (more reliable than clicking calendar)
    const dateInput = screen.getByRole('textbox', { name: /due date/i });
    await user.clear(dateInput);
    await user.type(dateInput, '12/25/2024');

    // fill required field and submit
    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'Holiday task'
    );
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(handleSubmit).toHaveBeenCalledWith(
      expect.objectContaining({
        dueDate: expect.any(Date),
      })
    );
  });

  it('handles invalid date input', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const dateInput = screen.getByRole('textbox', { name: /due date/i });
    await user.type(dateInput, 'invalid-date');

    // check for error state
    expect(dateInput).toHaveAttribute('aria-invalid', 'true');
  });

  it('allows submitting without a due date', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'Task without date'
    );
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(handleSubmit).toHaveBeenCalledWith(
      expect.objectContaining({ dueDate: null })
    );
  });
});
```

### testing MUI autocomplete

```typescript
describe('TodoForm - Autocomplete', () => {
  it('shows suggestions when typing', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });
    await user.click(assigneeInput);
    await user.type(assigneeInput, 'Al');

    // wait for filtered options to appear
    expect(await screen.findByRole('option', { name: /alice/i })).toBeInTheDocument();
  });

  it('selects an option from autocomplete dropdown', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    // open autocomplete and select option
    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });
    await user.click(assigneeInput);
    await user.click(await screen.findByRole('option', { name: /bob/i }));

    // verify selection
    expect(assigneeInput).toHaveValue('Bob');

    // submit form
    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'Task for Bob'
    );
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(handleSubmit).toHaveBeenCalledWith(
      expect.objectContaining({ assignee: 'Bob' })
    );
  });

  it('allows clearing the autocomplete selection', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });

    // select an option
    await user.click(assigneeInput);
    await user.click(await screen.findByRole('option', { name: /charlie/i }));
    expect(assigneeInput).toHaveValue('Charlie');

    // clear the selection
    const clearButton = screen.getByRole('button', { name: /clear/i });
    await user.click(clearButton);

    expect(assigneeInput).toHaveValue('');
  });

  it('filters options based on input', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });
    await user.type(assigneeInput, 'Di');

    // only Diana should match
    const options = await screen.findAllByRole('option');
    expect(options).toHaveLength(1);
    expect(options[0]).toHaveTextContent('Diana');
  });

  it('allows keyboard navigation of options', async () => {
    const user = userEvent.setup();
    render(<TodoForm onSubmit={vi.fn()} />);

    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });
    await user.click(assigneeInput);

    // wait for options to appear
    await screen.findByRole('option', { name: /alice/i });

    // navigate with keyboard
    await user.keyboard('{ArrowDown}{ArrowDown}{Enter}');

    // second option (Bob) should be selected
    expect(assigneeInput).toHaveValue('Bob');
  });
});
```

### testing complete form submission

```typescript
describe('TodoForm - Complete Submission', () => {
  it('submits form with all fields filled', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();

    render(<TodoForm onSubmit={handleSubmit} />);

    // fill title
    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'Complete project'
    );

    // fill date
    const dateInput = screen.getByRole('textbox', { name: /due date/i });
    await user.clear(dateInput);
    await user.type(dateInput, '01/15/2025');

    // select assignee
    const assigneeInput = screen.getByRole('combobox', { name: /assignee/i });
    await user.click(assigneeInput);
    await user.click(await screen.findByRole('option', { name: /diana/i }));

    // submit
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(handleSubmit).toHaveBeenCalledWith({
      title: 'Complete project',
      dueDate: expect.any(Date),
      assignee: 'Diana',
    });
  });
});
```

---

## 3. testing api calls

> when mocking fetch directly, `vi.stubGlobal('fetch', vi.fn()...)` is more robust
> than `global.fetch = vi.fn()` because it integrates with vitest's
> `unstubAllGlobals` config option for automatic cleanup.

### basic fetch mocking

```typescript
import { vi, describe, it, expect, beforeEach } from 'vitest';

describe('TodoList with API', () => {
  beforeEach(() => {
    vi.resetAllMocks();
  });

  it('displays todos fetched from API', async () => {
    const mockTodos = [
      { id: '1', title: 'Buy milk', completed: false },
      { id: '2', title: 'Walk dog', completed: true },
    ];

    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve(mockTodos),
    });

    render(<TodoList />);

    // wait for todos to load
    expect(await screen.findByText('Buy milk')).toBeInTheDocument();
    expect(screen.getByText('Walk dog')).toBeInTheDocument();
  });

  it('displays error message when fetch fails', async () => {
    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: false,
      status: 500,
    });

    render(<TodoList />);

    expect(await screen.findByText(/failed to load todos/i)).toBeInTheDocument();
  });

  it('displays loading state while fetching', async () => {
    let resolvePromise: (value: unknown) => void;
    const pendingPromise = new Promise((resolve) => {
      resolvePromise = resolve;
    });

    global.fetch = vi.fn().mockReturnValueOnce(pendingPromise);

    render(<TodoList />);

    // check loading state
    expect(screen.getByRole('progressbar')).toBeInTheDocument();

    // resolve the promise
    resolvePromise!({ ok: true, json: () => Promise.resolve([]) });

    // wait for loading to complete
    await waitForElementToBeRemoved(() => screen.queryByRole('progressbar'));
  });
});
```

### testing POST requests

```typescript
describe('Creating todos via API', () => {
  it('sends correct data when creating a todo', async () => {
    const user = userEvent.setup();
    const mockFetch = vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve([]), // initial GET
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({
          id: '3',
          title: 'New todo',
          completed: false,
        }), // POST response
      });

    global.fetch = mockFetch;

    render(<TodoApp />);

    // wait for initial load
    await screen.findByRole('textbox', { name: /task title/i });

    // fill and submit form
    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'New todo'
    );
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    // verify API was called correctly
    expect(mockFetch).toHaveBeenCalledWith('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title: 'New todo', completed: false }),
    });
  });

  it('shows newly created todo in the list', async () => {
    const user = userEvent.setup();
    global.fetch = vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve([]),
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({
          id: '1',
          title: 'Brand new todo',
          completed: false,
        }),
      });

    render(<TodoApp />);
    await screen.findByRole('textbox', { name: /task title/i });

    await user.type(
      screen.getByRole('textbox', { name: /task title/i }),
      'Brand new todo'
    );
    await user.click(screen.getByRole('button', { name: /add todo/i }));

    expect(await screen.findByText('Brand new todo')).toBeInTheDocument();
  });
});
```

---

## 4. testing custom hooks

### using renderHook

React Testing Library provides `renderHook` for testing custom hooks in isolation.

```typescript
import { renderHook, act, waitFor } from '@testing-library/react';
import { vi, describe, it, expect } from 'vitest';
```

### example: simple state hook

```typescript
// hooks/useCounter.ts
export const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
};
```

```typescript
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value of 0', () => {
    const { result } = renderHook(() => useCounter());

    expect(result.current.count).toBe(0);
  });

  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));

    expect(result.current.count).toBe(10);
  });

  it('increments the count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements the count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
    });

    expect(result.current.count).toBe(12);

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });
});
```

### example: todo list hook with CRUD operations

```typescript
// hooks/useTodos.ts
interface Todo {
  id: string;
  title: string;
  completed: boolean;
  dueDate: Date | null;
}

export const useTodos = (initialTodos: Todo[] = []) => {
  const [todos, setTodos] = useState<Todo[]>(initialTodos);

  const addTodo = (title: string, dueDate: Date | null = null) => {
    const newTodo: Todo = {
      id: crypto.randomUUID(),
      title,
      completed: false,
      dueDate,
    };
    setTodos((prev) => [...prev, newTodo]);
    return newTodo;
  };

  const toggleTodo = (id: string) => {
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id: string) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  };

  const completedCount = todos.filter((t) => t.completed).length;
  const pendingCount = todos.filter((t) => !t.completed).length;

  return { todos, addTodo, toggleTodo, deleteTodo, completedCount, pendingCount };
};
```

```typescript
// hooks/useTodos.test.ts
import { renderHook, act } from '@testing-library/react';
import { useTodos } from './useTodos';

describe('useTodos', () => {
  it('initializes with empty array by default', () => {
    const { result } = renderHook(() => useTodos());

    expect(result.current.todos).toEqual([]);
    expect(result.current.completedCount).toBe(0);
    expect(result.current.pendingCount).toBe(0);
  });

  it('initializes with provided todos', () => {
    const initialTodos = [
      { id: '1', title: 'Test', completed: false, dueDate: null },
    ];
    const { result } = renderHook(() => useTodos(initialTodos));

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.pendingCount).toBe(1);
  });

  it('adds a new todo', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('Buy groceries');
    });

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.todos[0].title).toBe('Buy groceries');
    expect(result.current.todos[0].completed).toBe(false);
  });

  it('adds a todo with due date', () => {
    const { result } = renderHook(() => useTodos());
    const dueDate = new Date('2025-01-15');

    act(() => {
      result.current.addTodo('Meeting', dueDate);
    });

    expect(result.current.todos[0].dueDate).toEqual(dueDate);
  });

  it('toggles todo completion status', () => {
    const initialTodos = [
      { id: '1', title: 'Test', completed: false, dueDate: null },
    ];
    const { result } = renderHook(() => useTodos(initialTodos));

    act(() => {
      result.current.toggleTodo('1');
    });

    expect(result.current.todos[0].completed).toBe(true);
    expect(result.current.completedCount).toBe(1);
    expect(result.current.pendingCount).toBe(0);
  });

  it('deletes a todo', () => {
    const initialTodos = [
      { id: '1', title: 'Test 1', completed: false, dueDate: null },
      { id: '2', title: 'Test 2', completed: false, dueDate: null },
    ];
    const { result } = renderHook(() => useTodos(initialTodos));

    act(() => {
      result.current.deleteTodo('1');
    });

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.todos[0].id).toBe('2');
  });
});
```

### testing hooks with api calls

```typescript
// hooks/useFetchTodos.ts
export const useFetchTodos = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchTodos = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/todos');
      if (!response.ok) throw new Error('Failed to fetch');
      const data = await response.json();
      setTodos(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchTodos();
  }, [fetchTodos]);

  return { todos, loading, error, refetch: fetchTodos };
};
```

```typescript
// hooks/useFetchTodos.test.ts
import { renderHook, waitFor, act } from '@testing-library/react';
import { vi, describe, it, expect, beforeEach } from 'vitest';
import { useFetchTodos } from './useFetchTodos';

describe('useFetchTodos', () => {
  beforeEach(() => {
    vi.resetAllMocks();
  });

  it('fetches todos on mount', async () => {
    const mockTodos = [{ id: '1', title: 'Test', completed: false }];

    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve(mockTodos),
    });

    const { result } = renderHook(() => useFetchTodos());

    // initially loading
    expect(result.current.loading).toBe(true);

    // wait for fetch to complete
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.todos).toEqual(mockTodos);
    expect(result.current.error).toBeNull();
  });

  it('handles fetch error', async () => {
    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: false,
      status: 500,
    });

    const { result } = renderHook(() => useFetchTodos());

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toBe('Failed to fetch');
    expect(result.current.todos).toEqual([]);
  });

  it('handles network error', async () => {
    global.fetch = vi.fn().mockRejectedValueOnce(new Error('Network error'));

    const { result } = renderHook(() => useFetchTodos());

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toBe('Network error');
  });

  it('refetches when refetch is called', async () => {
    const mockTodos1 = [{ id: '1', title: 'First', completed: false }];
    const mockTodos2 = [{ id: '2', title: 'Second', completed: true }];

    global.fetch = vi
      .fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockTodos1),
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockTodos2),
      });

    const { result } = renderHook(() => useFetchTodos());

    await waitFor(() => {
      expect(result.current.todos).toEqual(mockTodos1);
    });

    await act(async () => {
      await result.current.refetch();
    });

    expect(result.current.todos).toEqual(mockTodos2);
  });
});
```

### testing hooks that require providers

```typescript
// hooks/useTheme.ts (requires ThemeProvider context)
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};
```

```typescript
// hooks/useTheme.test.ts
import { renderHook } from '@testing-library/react';
import { ThemeProvider } from '@mui/material/styles';
import { createTheme } from '@mui/material';
import { useTheme } from './useTheme';

const theme = createTheme();

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <ThemeProvider theme={theme}>{children}</ThemeProvider>
);

describe('useTheme', () => {
  it('returns theme when wrapped in provider', () => {
    const { result } = renderHook(() => useTheme(), { wrapper });

    expect(result.current).toBeDefined();
    expect(result.current.palette).toBeDefined();
  });

  it('throws error when used outside provider', () => {
    // suppress console.error for this test
    const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});

    expect(() => {
      renderHook(() => useTheme());
    }).toThrow('useTheme must be used within ThemeProvider');

    consoleSpy.mockRestore();
  });
});
```

### testing hooks with debounce

```typescript
// hooks/useDebouncedSearch.ts
export const useDebouncedSearch = (delay = 300) => {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(query);
    }, delay);

    return () => clearTimeout(timer);
  }, [query, delay]);

  return { query, setQuery, debouncedQuery };
};
```

```typescript
// hooks/useDebouncedSearch.test.ts
import { renderHook, act } from '@testing-library/react';
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';
import { useDebouncedSearch } from './useDebouncedSearch';

describe('useDebouncedSearch', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('updates query immediately', () => {
    const { result } = renderHook(() => useDebouncedSearch());

    act(() => {
      result.current.setQuery('test');
    });

    expect(result.current.query).toBe('test');
    expect(result.current.debouncedQuery).toBe(''); // not updated yet
  });

  it('updates debouncedQuery after delay', () => {
    const { result } = renderHook(() => useDebouncedSearch(300));

    act(() => {
      result.current.setQuery('search term');
    });

    // before delay
    expect(result.current.debouncedQuery).toBe('');

    // after delay
    act(() => {
      vi.advanceTimersByTime(300);
    });

    expect(result.current.debouncedQuery).toBe('search term');
  });

  it('cancels previous debounce on rapid input', () => {
    const { result } = renderHook(() => useDebouncedSearch(300));

    act(() => {
      result.current.setQuery('a');
    });

    act(() => {
      vi.advanceTimersByTime(100);
    });

    act(() => {
      result.current.setQuery('ab');
    });

    act(() => {
      vi.advanceTimersByTime(100);
    });

    act(() => {
      result.current.setQuery('abc');
    });

    // 200ms since last setQuery('abc'), 100ms short of the 300ms delay
    act(() => {
      vi.advanceTimersByTime(200);
    });

    expect(result.current.debouncedQuery).toBe(''); // still waiting

    act(() => {
      vi.advanceTimersByTime(100);
    });

    expect(result.current.debouncedQuery).toBe('abc'); // 300ms since last input
  });

  it('uses custom delay', () => {
    const { result } = renderHook(() => useDebouncedSearch(500));

    act(() => {
      result.current.setQuery('test');
    });

    act(() => {
      vi.advanceTimersByTime(300);
    });

    expect(result.current.debouncedQuery).toBe(''); // still waiting

    act(() => {
      vi.advanceTimersByTime(200);
    });

    expect(result.current.debouncedQuery).toBe('test');
  });
});
```

### testing hooks with changing props (rerender)

```typescript
// hooks/useFilter.ts
export const useFilter = <T>(items: T[], filterFn: (item: T) => boolean) => {
  return useMemo(() => items.filter(filterFn), [items, filterFn]);
};
```

```typescript
// hooks/useFilter.test.ts
import { renderHook } from '@testing-library/react';
import { useFilter } from './useFilter';

describe('useFilter', () => {
  const todos = [
    { id: '1', title: 'Task 1', completed: true },
    { id: '2', title: 'Task 2', completed: false },
    { id: '3', title: 'Task 3', completed: true },
  ];

  it('filters items based on provided function', () => {
    const { result } = renderHook(() =>
      useFilter(todos, (todo) => todo.completed)
    );

    expect(result.current).toHaveLength(2);
    expect(result.current.map((t) => t.id)).toEqual(['1', '3']);
  });

  it('updates when items change', () => {
    const { result, rerender } = renderHook(
      ({ items }) => useFilter(items, (todo) => todo.completed),
      { initialProps: { items: todos } }
    );

    expect(result.current).toHaveLength(2);

    const newTodos = [
      ...todos,
      { id: '4', title: 'Task 4', completed: true },
    ];

    rerender({ items: newTodos });

    expect(result.current).toHaveLength(3);
  });

  it('updates when filter function changes', () => {
    // note: passing stable function references here matters because useFilter
    // uses useMemo with filterFn as a dependency. if you pass inline arrow
    // functions, useMemo recomputes every render regardless (new reference each time).
    const completedFilter = (todo: typeof todos[0]) => todo.completed;
    const pendingFilter = (todo: typeof todos[0]) => !todo.completed;

    const { result, rerender } = renderHook(
      ({ filterFn }) => useFilter(todos, filterFn),
      { initialProps: { filterFn: completedFilter } }
    );

    expect(result.current).toHaveLength(2);

    rerender({ filterFn: pendingFilter });

    expect(result.current).toHaveLength(1);
    expect(result.current[0].id).toBe('2');
  });
});
```

### key points for testing hooks

| Concept | Usage |
|---------|-------|
| `renderHook` | Renders a hook in isolation without a component |
| `result.current` | Access the current return value of the hook |
| `act()` | Wrap state updates to ensure React processes them |
| `waitFor` | Wait for async operations to complete |
| `rerender` | Re-render with new props to test prop changes |
| `wrapper` | Provide context providers the hook depends on |

---

## 5. common pitfalls and solutions

### problem: element not found after state update

```typescript
// BAD: query runs before React updates
it('shows success message', async () => {
  const user = userEvent.setup();
  render(<TodoForm onSubmit={vi.fn()} />);

  await user.click(screen.getByRole('button', { name: /submit/i }));
  expect(screen.getByText(/success/i)).toBeInTheDocument(); // may fail!
});

// GOOD: use findBy for async updates
it('shows success message', async () => {
  const user = userEvent.setup();
  render(<TodoForm onSubmit={vi.fn()} />);

  await user.click(screen.getByRole('button', { name: /submit/i }));
  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

### problem: testing MUI portaled components (dialogs, menus, popovers)

```typescript
// MUI renders these outside the component tree via portals
// always use screen queries, not within()

it('opens delete confirmation dialog', async () => {
  const user = userEvent.setup();
  render(<TodoItem todo={{ id: '1', title: 'Test' }} />);

  await user.click(screen.getByRole('button', { name: /delete/i }));

  // dialog is portaled to document.body - use screen, not within()
  expect(screen.getByRole('dialog')).toBeInTheDocument();
  expect(screen.getByText(/are you sure/i)).toBeInTheDocument();
});
```

### problem: act warnings

```typescript
// warning: an update was not wrapped in act(...)

// GOOD: use userEvent (automatically wraps in act)
const user = userEvent.setup();
await user.click(button);

// GOOD: use waitFor for async state updates
await waitFor(() => {
  expect(screen.getByText('Updated')).toBeInTheDocument();
});

// GOOD: wait for element to disappear
await waitForElementToBeRemoved(() => screen.queryByRole('progressbar'));
```

### problem: testing debounced input

```typescript
import { vi } from 'vitest';

it('searches after debounce delay', async () => {
  vi.useFakeTimers();
  const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });

  render(<SearchableTodoList />);

  await user.type(screen.getByRole('searchbox'), 'milk');

  // fast-forward past debounce delay
  await vi.advanceTimersByTimeAsync(300);

  expect(await screen.findByText('Buy milk')).toBeInTheDocument();

  vi.useRealTimers();
});
```

### problem: multiple elements with same role

```typescript
// BAD: ambiguous query
screen.getByRole('button'); // fails if multiple buttons exist

// GOOD: use name option
screen.getByRole('button', { name: /submit/i });

// GOOD: use within() to scope queries
const todoItem = screen.getByRole('listitem', { name: /buy milk/i });
within(todoItem).getByRole('button', { name: /delete/i });

// GOOD: use getAllByRole and filter
const deleteButtons = screen.getAllByRole('button', { name: /delete/i });
expect(deleteButtons).toHaveLength(3);
```

### problem: testing select/dropdown changes

```typescript
// for MUI Select components
it('changes priority selection', async () => {
  const user = userEvent.setup();
  render(<TodoForm onSubmit={vi.fn()} />);

  // open the select
  await user.click(screen.getByRole('combobox', { name: /priority/i }));

  // click the option (renders in a portal)
  await user.click(screen.getByRole('option', { name: /high/i }));

  // verify the select shows the new value
  expect(screen.getByRole('combobox', { name: /priority/i })).toHaveTextContent('High');
});
```

### problem: flaky tests due to timing

```typescript
// BAD: arbitrary timeouts
await new Promise((r) => setTimeout(r, 1000));

// GOOD: wait for specific conditions
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});

// GOOD: use findBy with custom timeout
await screen.findByText('Loaded', {}, { timeout: 3000 });

// GOOD: wait for element to be removed
await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));
```

### quick reference: common role names

| Element Type | Role |
|--------------|------|
| Button, MUI Button | `button` |
| Input, MUI TextField | `textbox` |
| MUI Select | `combobox` |
| Checkbox, MUI Checkbox | `checkbox` |
| Radio, MUI Radio | `radio` |
| MUI Switch | `checkbox` (verify for your MUI version) |
| MUI Dialog | `dialog` |
| Menu | `menu` |
| MenuItem | `menuitem` |
| Tab | `tab` |
| Alert, MUI Alert | `alert` |
| MUI CircularProgress | `progressbar` |
| Autocomplete options | `option` |
| List | `list` |
| ListItem (with text) | `listitem` |
| Link | `link` |
| Heading (h1-h6) | `heading` |
| Image | `img` |

### debugging tips

```typescript
// print the current DOM state
screen.debug();

// print specific element
screen.debug(screen.getByRole('form'));

// limit debug output length
screen.debug(undefined, 30000);

// log accessible roles in container
import { logRoles } from '@testing-library/react';
const { container } = render(<MyComponent />);
logRoles(container);

// generate testing-playground URL for query help
screen.logTestingPlaygroundURL();
```

---

## quick command reference

```bash
# run all tests
yarn test

# run tests in watch mode
yarn test --watch

# run specific test file
yarn test TodoList.test.tsx

# run tests matching pattern
yarn test -t "creates todo"

# run with coverage
yarn test --coverage

# run with verbose output
yarn test --reporter=verbose

# update snapshots
yarn test -u
```
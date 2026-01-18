# MVVM Architecture Pattern

## Table of Contents
- [Introduction](#introduction)
- [Architecture Components](#architecture-components)
- [Data Flow](#data-flow)
- [Implementation in React/TypeScript](#implementation-in-reacttypescript)
- [Benefits and Limitations](#benefits-and-limitations)
- [Common Patterns](#common-patterns)
- [Anti-Patterns](#anti-patterns)
- [Comparison with Other Architectures](#comparison-with-other-architectures)
- [Why MVVM](#why-mvvm)

---

## Introduction

Model-View-ViewModel (MVVM) is an architectural pattern that separates application concerns into three distinct layers. Originally developed by Microsoft for WPF and Silverlight applications, MVVM has been adapted for modern web development frameworks including React.

The primary goal of MVVM is to achieve separation of concerns by decoupling the user interface from business logic and data management. This separation improves testability, maintainability, and enables parallel development of UI and business logic.

### Core Principle

MVVM enforces a clear data flow between the View and ViewModel, eliminating the need for the View to directly manipulate business logic or data structures.

---

## Architecture Components

### Model

The Model represents the data layer of the application. It encapsulates:

- Domain entities and business objects
- Data structures and types
- Data validation rules
- Business logic unrelated to presentation

**Responsibilities:**
- Define data structures
- Enforce business rules
- Provide data access interfaces
- Emit notifications when data changes

**Key characteristics:**
- No knowledge of View or ViewModel
- Framework-agnostic
- Reusable across different interfaces

**Example:**

```typescript
// Model: Todo domain entity
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
  priority: 'low' | 'medium' | 'high';
}

// Model: Validation logic
export class TodoValidator {
  static validate(todo: Partial<Todo>): string[] {
    const errors: string[] = [];
    
    if (!todo.text || todo.text.trim().length === 0) {
      errors.push('Todo text cannot be empty');
    }
    
    if (todo.text && todo.text.length > 500) {
      errors.push('Todo text cannot exceed 500 characters');
    }
    
    return errors;
  }
}

// Model: Business logic
export class TodoService {
  private apiUrl = '/api/todos';
  
  async fetchTodos(): Promise<Todo[]> {
    const response = await fetch(this.apiUrl);
    return response.json();
  }
  
  async createTodo(todo: Omit<Todo, 'id' | 'createdAt'>): Promise<Todo> {
    const response = await fetch(this.apiUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(todo),
    });
    return response.json();
  }
  
  async updateTodo(id: string, updates: Partial<Todo>): Promise<Todo> {
    const response = await fetch(`${this.apiUrl}/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates),
    });
    return response.json();
  }
  
  async deleteTodo(id: string): Promise<void> {
    await fetch(`${this.apiUrl}/${id}`, { method: 'DELETE' });
  }
}
```

---

### ViewModel

The ViewModel serves as an intermediary between the View and Model. It transforms Model data into a format suitable for display and handles user interactions.

**Responsibilities:**
- Expose data in a View-friendly format
- Handle user input and commands
- Orchestrate Model operations
- Manage presentation state (loading, errors, etc.)
- Provide data bindings for the View

**Key characteristics:**
- No direct references to UI components
- Contains presentation logic, not business logic
- Testable without UI framework
- May aggregate data from multiple Models

In React, ViewModels are implemented as custom hooks. This approach leverages React's built-in state management and provides excellent TypeScript integration.

**Example:**

```typescript
// ViewModel: Todo list management as a custom hook
import { useState, useEffect, useMemo, useCallback } from 'react';

export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');
  
  const todoService = useMemo(() => new TodoService(), []);
  
  // Computed properties for View
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  }, [todos, filter]);
  
  const activeCount = useMemo(() => {
    return todos.filter(todo => !todo.completed).length;
  }, [todos]);
  
  const completedCount = useMemo(() => {
    return todos.filter(todo => todo.completed).length;
  }, [todos]);
  
  // Commands for View to invoke
  const loadTodos = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const data = await todoService.fetchTodos();
      setTodos(data);
    } catch (err) {
      setError('Failed to load todos');
      console.error(err);
    } finally {
      setLoading(false);
    }
  }, [todoService]);
  
  const addTodo = useCallback(async (text: string, priority: Todo['priority']) => {
    const errors = TodoValidator.validate({ text });
    if (errors.length > 0) {
      setError(errors[0]);
      return;
    }
    
    try {
      const newTodo = await todoService.createTodo({
        text,
        completed: false,
        priority,
      });
      setTodos(prev => [...prev, newTodo]);
      setError(null);
    } catch (err) {
      setError('Failed to create todo');
      console.error(err);
    }
  }, [todoService]);
  
  const toggleTodo = useCallback(async (id: string) => {
    const todo = todos.find(t => t.id === id);
    if (!todo) return;
    
    try {
      const updated = await todoService.updateTodo(id, {
        completed: !todo.completed,
      });
      
      setTodos(prev => prev.map(t => t.id === id ? updated : t));
    } catch (err) {
      setError('Failed to update todo');
      console.error(err);
    }
  }, [todos, todoService]);
  
  const removeTodo = useCallback(async (id: string) => {
    try {
      await todoService.deleteTodo(id);
      setTodos(prev => prev.filter(t => t.id !== id));
    } catch (err) {
      setError('Failed to delete todo');
      console.error(err);
    }
  }, [todoService]);
  
  const handleFilterChange = useCallback((newFilter: 'all' | 'active' | 'completed') => {
    setFilter(newFilter);
  }, []);
  
  // Return ViewModel interface
  return {
    // State
    todos: filteredTodos,
    loading,
    error,
    filter,
    activeCount,
    completedCount,
    
    // Commands
    loadTodos,
    addTodo,
    toggleTodo,
    removeTodo,
    handleFilterChange,
  };
}
```

---

### View

The View represents the user interface layer. It displays data provided by the ViewModel and routes user interactions back to the ViewModel.

**Responsibilities:**
- Render UI components
- Display ViewModel data
- Capture user input
- Invoke ViewModel commands
- Respond to ViewModel state changes

**Key characteristics:**
- Contains minimal logic (primarily rendering)
- No direct access to Model
- Observes ViewModel for changes
- Platform-specific (React components)

**Example:**

```typescript
// View: React component using the ViewModel
import { useEffect } from 'react';
import { useTodoListViewModel } from './useTodoListViewModel';

export function TodoListView() {
  const viewModel = useTodoListViewModel();
  
  // Load todos on mount
  useEffect(() => {
    viewModel.loadTodos();
  }, [viewModel.loadTodos]);
  
  const handleAddTodo = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const text = formData.get('text') as string;
    const priority = formData.get('priority') as Todo['priority'];
    
    await viewModel.addTodo(text, priority);
    e.currentTarget.reset();
  };
  
  if (viewModel.loading) {
    return <div>Loading todos...</div>;
  }
  
  return (
    <div className="todo-list">
      <h1>Todo List</h1>
      
      {viewModel.error && (
        <div className="error">{viewModel.error}</div>
      )}
      
      <form onSubmit={handleAddTodo}>
        <input
          type="text"
          name="text"
          placeholder="What needs to be done?"
          required
        />
        <select name="priority" defaultValue="medium">
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="high">High</option>
        </select>
        <button type="submit">Add</button>
      </form>
      
      <div className="filters">
        <button
          className={viewModel.filter === 'all' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('all')}
        >
          All ({viewModel.todos.length})
        </button>
        <button
          className={viewModel.filter === 'active' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('active')}
        >
          Active ({viewModel.activeCount})
        </button>
        <button
          className={viewModel.filter === 'completed' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('completed')}
        >
          Completed ({viewModel.completedCount})
        </button>
      </div>
      
      <ul>
        {viewModel.todos.map(todo => (
          <li key={todo.id} className={todo.completed ? 'completed' : ''}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => viewModel.toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
            <span className="priority">{todo.priority}</span>
            <button onClick={() => viewModel.removeTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
      
      <div className="stats">
        {viewModel.activeCount} item{viewModel.activeCount !== 1 ? 's' : ''} remaining
      </div>
    </div>
  );
}
```

---

## Data Flow

### Unidirectional Flow in React

```
User Action → View → ViewModel Command → Model Update → ViewModel State Change → View Re-render
```

**Sequence:**
1. User interacts with View (click, input, etc.)
2. View invokes ViewModel command (via hook return value)
3. ViewModel processes command and updates Model
4. Model returns updated data
5. ViewModel updates its state (via useState)
6. React re-renders View with new data

This unidirectional flow makes applications predictable and easier to debug.

---

## Implementation in React/TypeScript

### Standard Approach: Custom Hooks as ViewModels

ViewModels are implemented as custom hooks. This approach provides:
- Native React state management integration
- Excellent TypeScript support
- Automatic re-rendering on state changes
- Easy testing with `@testing-library/react-hooks`

**ViewModel Hook Structure:**

```typescript
// src/features/todos/viewmodels/useTodoListViewModel.ts
import { useState, useEffect, useMemo, useCallback } from 'react';
import { useAppDispatch, useAppSelector } from '@app/store/hooks';
import { fetchTodos, selectFilteredTodos, selectTodosLoading } from '../models/store/todoSlice';
import { todoService } from '../models/services/todoService';
import type { Todo, TodoFilter } from '../types/todo.types';

export function useTodoListViewModel() {
  // State from Model layer (Redux)
  const dispatch = useAppDispatch();
  const todos = useAppSelector(selectFilteredTodos);
  const loading = useAppSelector(selectTodosLoading);
  
  // ViewModel-specific state (local)
  const [filter, setFilter] = useState<TodoFilter>('all');
  const [searchTerm, setSearchTerm] = useState('');
  
  // Computed properties for View
  const filteredTodos = useMemo(() => {
    let result = todos;
    
    // Apply filter
    if (filter === 'active') {
      result = result.filter(todo => !todo.completed);
    } else if (filter === 'completed') {
      result = result.filter(todo => todo.completed);
    }
    
    // Apply search
    if (searchTerm) {
      result = result.filter(todo =>
        todo.text.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    return result;
  }, [todos, filter, searchTerm]);
  
  const activeCount = useMemo(() => {
    return todos.filter(t => !t.completed).length;
  }, [todos]);
  
  const completedCount = useMemo(() => {
    return todos.filter(t => t.completed).length;
  }, [todos]);
  
  const completionPercentage = useMemo(() => {
    if (todos.length === 0) return 0;
    return Math.round((completedCount / todos.length) * 100);
  }, [todos.length, completedCount]);
  
  // Commands for View to invoke
  const loadTodos = useCallback(() => {
    dispatch(fetchTodos());
  }, [dispatch]);
  
  const handleFilterChange = useCallback((newFilter: TodoFilter) => {
    setFilter(newFilter);
  }, []);
  
  const handleSearchChange = useCallback((term: string) => {
    setSearchTerm(term);
  }, []);
  
  const deleteTodo = useCallback(async (todoId: string) => {
    await todoService.deleteTodo(todoId);
    dispatch(fetchTodos()); // Refresh list
  }, [dispatch]);
  
  // Load data on mount
  useEffect(() => {
    loadTodos();
  }, [loadTodos]);
  
  // Return ViewModel interface
  return {
    // State
    todos: filteredTodos,
    loading,
    filter,
    searchTerm,
    activeCount,
    completedCount,
    completionPercentage,
    
    // Commands
    loadTodos,
    handleFilterChange,
    handleSearchChange,
    deleteTodo,
  };
}
```

**View Component Using ViewModel:**

```typescript
// src/features/todos/components/TodoList/TodoList.tsx
import { useTodoListViewModel } from '../../viewmodels/useTodoListViewModel';
import { LoadingSpinner } from '@shared/components/LoadingSpinner';
import { TodoItem } from '../TodoItem';

export function TodoList() {
  const viewModel = useTodoListViewModel();
  
  if (viewModel.loading) {
    return <LoadingSpinner />;
  }
  
  return (
    <div className="todo-list">
      <div className="todo-list__search">
        <input
          type="text"
          value={viewModel.searchTerm}
          onChange={(e) => viewModel.handleSearchChange(e.target.value)}
          placeholder="Search todos..."
        />
      </div>
      
      <div className="todo-list__filters">
        <button
          className={viewModel.filter === 'all' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('all')}
        >
          All ({viewModel.todos.length})
        </button>
        <button
          className={viewModel.filter === 'active' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('active')}
        >
          Active ({viewModel.activeCount})
        </button>
        <button
          className={viewModel.filter === 'completed' ? 'active' : ''}
          onClick={() => viewModel.handleFilterChange('completed')}
        >
          Completed ({viewModel.completedCount})
        </button>
      </div>
      
      <div className="todo-list__progress">
        {viewModel.completionPercentage}% complete
      </div>
      
      <ul className="todo-list__items">
        {viewModel.todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onDelete={viewModel.deleteTodo}
          />
        ))}
      </ul>
    </div>
  );
}
```

### Form ViewModel Example

For complex forms, dedicated ViewModels should be created:

```typescript
// src/features/todos/viewmodels/useTodoFormViewModel.ts
import { useState, useCallback } from 'react';
import { todoService } from '../models/services/todoService';
import { TodoValidator } from '../models/validation/todoValidation';
import type { Todo } from '../types/todo.types';

interface TodoFormState {
  text: string;
  priority: Todo['priority'];
  description: string;
}

const initialState: TodoFormState = {
  text: '',
  priority: 'medium',
  description: '',
};

export function useTodoFormViewModel(onSuccess?: () => void) {
  const [formState, setFormState] = useState<TodoFormState>(initialState);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState<string | null>(null);
  
  // Field update handlers
  const updateField = useCallback(<K extends keyof TodoFormState>(
    field: K,
    value: TodoFormState[K]
  ) => {
    setFormState(prev => ({ ...prev, [field]: value }));
    // Clear field error on change
    setErrors(prev => {
      const next = { ...prev };
      delete next[field];
      return next;
    });
  }, []);
  
  // Validation
  const validate = useCallback((): boolean => {
    const validationErrors = TodoValidator.validate(formState);
    
    if (validationErrors.length > 0) {
      const errorMap: Record<string, string> = {};
      validationErrors.forEach(err => {
        // Simple mapping - in production, use structured errors
        if (err.includes('text')) {
          errorMap.text = err;
        }
      });
      setErrors(errorMap);
      return false;
    }
    
    setErrors({});
    return true;
  }, [formState]);
  
  // Form submission
  const submit = useCallback(async (): Promise<boolean> => {
    if (!validate()) {
      return false;
    }
    
    setIsSubmitting(true);
    setSubmitError(null);
    
    try {
      await todoService.createTodo({
        text: formState.text,
        priority: formState.priority,
        completed: false,
      });
      
      // Reset form on success
      setFormState(initialState);
      onSuccess?.();
      return true;
    } catch (err) {
      setSubmitError('Failed to create todo. Please try again.');
      return false;
    } finally {
      setIsSubmitting(false);
    }
  }, [formState, validate, onSuccess]);
  
  // Reset form
  const reset = useCallback(() => {
    setFormState(initialState);
    setErrors({});
    setSubmitError(null);
  }, []);
  
  // Computed state
  const isValid = Object.keys(errors).length === 0 && formState.text.trim().length > 0;
  
  return {
    // Form state
    formState,
    errors,
    isSubmitting,
    submitError,
    isValid,
    
    // Commands
    updateField,
    submit,
    reset,
  };
}
```

---

## Common Patterns

### 1. Computed Properties

Derived values that automatically update when dependencies change:

```typescript
export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const completionPercentage = useMemo(() => {
    if (todos.length === 0) return 0;
    const completed = todos.filter(t => t.completed).length;
    return Math.round((completed / todos.length) * 100);
  }, [todos]);
  
  const hasHighPriorityTodos = useMemo(() => {
    return todos.some(t => t.priority === 'high' && !t.completed);
  }, [todos]);
  
  return { todos, completionPercentage, hasHighPriorityTodos };
}
```

### 2. Command Pattern

Encapsulating user actions as methods:

```typescript
export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const clearCompleted = useCallback(async () => {
    const completedIds = todos
      .filter(t => t.completed)
      .map(t => t.id);
    
    await Promise.all(
      completedIds.map(id => todoService.deleteTodo(id))
    );
    
    setTodos(prev => prev.filter(t => !t.completed));
  }, [todos]);
  
  const toggleAll = useCallback(async () => {
    const allCompleted = todos.every(t => t.completed);
    const newStatus = !allCompleted;
    
    await Promise.all(
      todos.map(t => todoService.updateTodo(t.id, { completed: newStatus }))
    );
    
    setTodos(prev => prev.map(t => ({ ...t, completed: newStatus })));
  }, [todos]);
  
  return { todos, clearCompleted, toggleAll };
}
```

### 3. Loading and Error States

Managing asynchronous operation states:

```typescript
type OperationState = 'idle' | 'loading' | 'error' | 'success';

export function useTodoListViewModel() {
  const [operationState, setOperationState] = useState<OperationState>('idle');
  const [errorMessage, setErrorMessage] = useState<string | null>(null);
  
  const isLoading = operationState === 'loading';
  const isError = operationState === 'error';
  
  const executeOperation = useCallback(async <T>(
    operation: () => Promise<T>
  ): Promise<T | null> => {
    setOperationState('loading');
    setErrorMessage(null);
    
    try {
      const result = await operation();
      setOperationState('success');
      return result;
    } catch (err) {
      setOperationState('error');
      setErrorMessage(err instanceof Error ? err.message : 'Unknown error');
      return null;
    }
  }, []);
  
  return { isLoading, isError, errorMessage, executeOperation };
}
```

### 4. Optimistic Updates

Updating UI immediately while syncing with server:

```typescript
export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const toggleTodo = useCallback(async (id: string) => {
    // Optimistically update UI
    setTodos(prev => prev.map(t =>
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
    
    try {
      // Sync with server
      const todo = todos.find(t => t.id === id)!;
      await todoService.updateTodo(id, { completed: !todo.completed });
    } catch (err) {
      // Revert on error
      setTodos(prev => prev.map(t =>
        t.id === id ? { ...t, completed: !t.completed } : t
      ));
      console.error('Failed to toggle todo:', err);
    }
  }, [todos]);
  
  return { todos, toggleTodo };
}
```

---

## Anti-Patterns

### 1. View Logic in ViewModel

The ViewModel should not contain UI-specific logic:

```typescript
// Incorrect - ViewModel contains UI logic
export function useTodoListViewModel() {
  const deleteTodo = useCallback(async (id: string) => {
    const confirmed = window.confirm('Delete this todo?'); // UI logic in ViewModel
    if (!confirmed) return;
    
    await todoService.deleteTodo(id);
    setTodos(prev => prev.filter(t => t.id !== id));
  }, []);
  
  return { deleteTodo };
}

// Correct - View handles UI logic
export function useTodoListViewModel() {
  const deleteTodo = useCallback(async (id: string) => {
    await todoService.deleteTodo(id);
    setTodos(prev => prev.filter(t => t.id !== id));
  }, []);
  
  return { deleteTodo };
}

// In View component:
function TodoList() {
  const viewModel = useTodoListViewModel();
  
  const handleDelete = async (id: string) => {
    const confirmed = window.confirm('Delete this todo?');
    if (confirmed) {
      await viewModel.deleteTodo(id);
    }
  };
  
  return <div>...</div>;
}
```

### 2. Model References in View

The View should never directly access the Model:

```typescript
// Incorrect - View directly uses Model
export function TodoListView() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const todoService = new TodoService(); // Direct Model access
  
  useEffect(() => {
    todoService.fetchTodos().then(setTodos); // Bypassing ViewModel
  }, []);
  
  return <div>...</div>;
}

// Correct - View uses ViewModel
export function TodoListView() {
  const viewModel = useTodoListViewModel();
  
  useEffect(() => {
    viewModel.loadTodos(); // Through ViewModel
  }, [viewModel.loadTodos]);
  
  return <div>...</div>;
}
```

### 3. Business Logic in ViewModel

Complex business rules belong in the Model, not ViewModel:

```typescript
// Incorrect - Business logic in ViewModel
export function useTodoListViewModel() {
  const calculatePriorityScore = useCallback((todo: Todo): number => {
    // Complex business calculation should be in Model
    let score = 0;
    if (todo.priority === 'high') score += 10;
    if (todo.priority === 'medium') score += 5;
    if (!todo.completed) score += 3;
    const daysSinceCreation = (Date.now() - todo.createdAt.getTime()) / (1000 * 60 * 60 * 24);
    score += daysSinceCreation * 0.1;
    return score;
  }, []);
  
  return { calculatePriorityScore };
}

// Correct - Business logic in Model
// models/services/todoPriorityCalculator.ts
export class TodoPriorityCalculator {
  static calculateScore(todo: Todo): number {
    let score = 0;
    if (todo.priority === 'high') score += 10;
    if (todo.priority === 'medium') score += 5;
    if (!todo.completed) score += 3;
    const daysSinceCreation = (Date.now() - todo.createdAt.getTime()) / (1000 * 60 * 60 * 24);
    score += daysSinceCreation * 0.1;
    return score;
  }
}

// ViewModel uses Model for business logic
export function useTodoListViewModel() {
  const sortedByPriority = useMemo(() => {
    return [...todos].sort((a, b) =>
      TodoPriorityCalculator.calculateScore(b) - TodoPriorityCalculator.calculateScore(a)
    );
  }, [todos]);
  
  return { sortedByPriority };
}
```

### 4. God ViewModel

Avoid creating ViewModels that manage too many responsibilities:

```typescript
// Incorrect - Single ViewModel managing everything
export function useApplicationViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [user, setUser] = useState<User | null>(null);
  const [settings, setSettings] = useState<Settings>({});
  const [notifications, setNotifications] = useState<Notification[]>([]);
  // Too many concerns in one ViewModel
}

// Correct - Separate ViewModels for each concern
export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  // Only todo-related logic
}

export function useUserViewModel() {
  const [user, setUser] = useState<User | null>(null);
  // Only user-related logic
}

export function useNotificationViewModel() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  // Only notification-related logic
}
```

### 5. ViewModel Directly Manipulating DOM

ViewModels should return state, not manipulate the DOM:

```typescript
// Incorrect - ViewModel manipulates DOM
export function useTodoFormViewModel() {
  const submitForm = useCallback(() => {
    document.getElementById('submit-btn')?.classList.add('loading'); // DOM manipulation
    // Submit logic
  }, []);
  
  return { submitForm };
}

// Correct - ViewModel returns state, View handles DOM
export function useTodoFormViewModel() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const submitForm = useCallback(async () => {
    setIsSubmitting(true);
    // Submit logic
    setIsSubmitting(false);
  }, []);
  
  return { isSubmitting, submitForm };
}

// View uses state for styling
function TodoForm() {
  const { isSubmitting, submitForm } = useTodoFormViewModel();
  
  return (
    <button
      className={isSubmitting ? 'loading' : ''}
      onClick={submitForm}
      disabled={isSubmitting}
    >
      {isSubmitting ? 'Submitting...' : 'Submit'}
    </button>
  );
}
```

---

## Comparison with Other Architectures

### MVC (Model-View-Controller)

**Key Difference:** In MVC, the Controller handles user input and updates both Model and View. The View can directly observe the Model.

```
User → View → Controller → Model
                ↓           ↓
              View ← ← ← ← Model
```

**MVVM vs MVC:**
- MVVM: View binds to ViewModel; no direct Model access
- MVC: View can observe Model directly; Controller orchestrates updates
- MVVM better supports data binding and declarative UIs
- MVC more common in traditional server-rendered applications

### MVP (Model-View-Presenter)

**Key Difference:** In MVP, the Presenter mediates all communication between View and Model. The View is passive and has an interface the Presenter uses.

```
User → View → Presenter → Model
        ↑         ↓
        ← ← ← ← ←
```

**MVVM vs MVP:**
- MVVM: Data binding between View and ViewModel
- MVP: Presenter explicitly updates View through interface
- MVVM allows for more declarative Views
- MVP provides stricter separation and easier View testing

### Flux/Redux

**Key Difference:** Flux enforces unidirectional data flow with a central Store and Actions.

```
User → View → Action → Dispatcher → Store → View
```

**MVVM vs Flux:**
- MVVM: Component-level ViewModels with local state
- Flux: Global state management with centralized Store
- MVVM better for component isolation
- Flux better for complex state synchronization across components

**Note:** MVVM is often combined with Redux. The ViewModel hook can use Redux selectors for shared state while managing local presentation state internally.

---

## Why MVVM

MVVM is well-suited for modern React applications because:

1. **Presentation logic complexity justifies abstraction** - Forms often require complex validation, computed properties, and orchestration of multiple data sources

2. **Clear separation enables parallel development** - Frontend engineers can work on Views while others work on ViewModels and Models

3. **Comprehensive unit testing** - ViewModels are pure TypeScript functions that can be tested without rendering React components

4. **Multiple views can share ViewModels** - The same ViewModel logic can power different UI representations

5. **TypeScript integration** - Custom hooks provide excellent type inference and IDE support

6. **React native patterns** - Using hooks as ViewModels aligns with React's composition model and doesn't require additional libraries

The key insight is that ViewModels are implemented as custom hooks, not class-based structures. This provides the architectural benefits of MVVM while staying idiomatic to React.
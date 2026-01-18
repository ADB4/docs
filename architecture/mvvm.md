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
- [When to Use MVVM](#when-to-use-mvvm)

---

## Introduction

Model-View-ViewModel (MVVM) is an architectural pattern that separates application concerns into three distinct layers. Originally developed by Microsoft for WPF and Silverlight applications, MVVM has been adapted for modern web development frameworks including React, Vue, and Angular.

The primary goal of MVVM is to achieve separation of concerns by decoupling the user interface from business logic and data management. This separation improves testability, maintainability, and enables parallel development of UI and business logic.

### Core Principle

MVVM enforces unidirectional or bidirectional data binding between the View and ViewModel, eliminating the need for the View to directly manipulate business logic or data structures.

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

**Example:**

```typescript
// ViewModel: Todo list management
export class TodoListViewModel {
  private todoService: TodoService;
  private todos: Todo[] = [];
  private loading: boolean = false;
  private error: string | null = null;
  private filter: 'all' | 'active' | 'completed' = 'all';
  
  constructor(todoService: TodoService) {
    this.todoService = todoService;
  }
  
  // Computed properties for View
  get filteredTodos(): Todo[] {
    switch (this.filter) {
      case 'active':
        return this.todos.filter(todo => !todo.completed);
      case 'completed':
        return this.todos.filter(todo => todo.completed);
      default:
        return this.todos;
    }
  }
  
  get activeCount(): number {
    return this.todos.filter(todo => !todo.completed).length;
  }
  
  get completedCount(): number {
    return this.todos.filter(todo => todo.completed).length;
  }
  
  get isLoading(): boolean {
    return this.loading;
  }
  
  get errorMessage(): string | null {
    return this.error;
  }
  
  get currentFilter(): 'all' | 'active' | 'completed' {
    return this.filter;
  }
  
  // Commands for View to invoke
  async loadTodos(): Promise<void> {
    this.loading = true;
    this.error = null;
    
    try {
      this.todos = await this.todoService.fetchTodos();
    } catch (err) {
      this.error = 'Failed to load todos';
      console.error(err);
    } finally {
      this.loading = false;
    }
  }
  
  async addTodo(text: string, priority: Todo['priority']): Promise<void> {
    const errors = TodoValidator.validate({ text });
    if (errors.length > 0) {
      this.error = errors[0];
      return;
    }
    
    try {
      const newTodo = await this.todoService.createTodo({
        text,
        completed: false,
        priority,
      });
      this.todos.push(newTodo);
      this.error = null;
    } catch (err) {
      this.error = 'Failed to create todo';
      console.error(err);
    }
  }
  
  async toggleTodo(id: string): Promise<void> {
    const todo = this.todos.find(t => t.id === id);
    if (!todo) return;
    
    try {
      const updated = await this.todoService.updateTodo(id, {
        completed: !todo.completed,
      });
      
      const index = this.todos.findIndex(t => t.id === id);
      this.todos[index] = updated;
    } catch (err) {
      this.error = 'Failed to update todo';
      console.error(err);
    }
  }
  
  async removeTodo(id: string): Promise<void> {
    try {
      await this.todoService.deleteTodo(id);
      this.todos = this.todos.filter(t => t.id !== id);
    } catch (err) {
      this.error = 'Failed to delete todo';
      console.error(err);
    }
  }
  
  setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.filter = filter;
  }
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
- Platform-specific (React, Vue, Angular, etc.)

**Example:**

```typescript
// View: React component using the ViewModel
import { useState, useEffect } from 'react';
import { TodoListViewModel } from './TodoListViewModel';
import { TodoService } from './TodoService';

export function TodoListView() {
  // Initialize ViewModel
  const [viewModel] = useState(() => new TodoListViewModel(new TodoService()));
  const [, forceUpdate] = useState({});
  
  // Load todos on mount
  useEffect(() => {
    viewModel.loadTodos();
  }, [viewModel]);
  
  // Force re-render when ViewModel changes
  // In production, use a proper observable pattern or state management
  const handleViewModelChange = () => {
    forceUpdate({});
  };
  
  const handleAddTodo = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const text = formData.get('text') as string;
    const priority = formData.get('priority') as Todo['priority'];
    
    await viewModel.addTodo(text, priority);
    handleViewModelChange();
    e.currentTarget.reset();
  };
  
  const handleToggle = async (id: string) => {
    await viewModel.toggleTodo(id);
    handleViewModelChange();
  };
  
  const handleDelete = async (id: string) => {
    await viewModel.removeTodo(id);
    handleViewModelChange();
  };
  
  const handleFilterChange = (filter: 'all' | 'active' | 'completed') => {
    viewModel.setFilter(filter);
    handleViewModelChange();
  };
  
  if (viewModel.isLoading) {
    return <div>Loading todos...</div>;
  }
  
  return (
    <div className="todo-list">
      <h1>Todo List</h1>
      
      {viewModel.errorMessage && (
        <div className="error">{viewModel.errorMessage}</div>
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
          className={viewModel.currentFilter === 'all' ? 'active' : ''}
          onClick={() => handleFilterChange('all')}
        >
          All ({viewModel.filteredTodos.length})
        </button>
        <button
          className={viewModel.currentFilter === 'active' ? 'active' : ''}
          onClick={() => handleFilterChange('active')}
        >
          Active ({viewModel.activeCount})
        </button>
        <button
          className={viewModel.currentFilter === 'completed' ? 'active' : ''}
          onClick={() => handleFilterChange('completed')}
        >
          Completed ({viewModel.completedCount})
        </button>
      </div>
      
      <ul>
        {viewModel.filteredTodos.map(todo => (
          <li key={todo.id} className={todo.completed ? 'completed' : ''}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggle(todo.id)}
            />
            <span>{todo.text}</span>
            <span className="priority">{todo.priority}</span>
            <button onClick={() => handleDelete(todo.id)}>Delete</button>
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

### Unidirectional Flow (Recommended for React)

```
User Action → View → ViewModel Command → Model Update → ViewModel State Change → View Re-render
```

**Sequence:**
1. User interacts with View (click, input, etc.)
2. View invokes ViewModel command
3. ViewModel processes command and updates Model
4. Model returns updated data
5. ViewModel updates its state
6. View observes ViewModel change and re-renders

### Bidirectional Data Binding (Traditional MVVM)

```
View ↔ ViewModel ↔ Model
```

In frameworks with native data binding (Vue, Angular), changes in either View or ViewModel automatically synchronize with each other.

---

## Implementation in React/TypeScript

### Using React Hooks and Observable Pattern

For proper MVVM implementation in React, ViewModels should be observable. Here's an implementation using MobX:

```typescript
// ViewModel with MobX observables
import { makeAutoObservable } from 'mobx';
import { TodoService } from './TodoService';
import { Todo } from './Todo';

export class TodoListViewModel {
  todos: Todo[] = [];
  loading: boolean = false;
  error: string | null = null;
  filter: 'all' | 'active' | 'completed' = 'all';
  
  constructor(private todoService: TodoService) {
    makeAutoObservable(this);
  }
  
  get filteredTodos(): Todo[] {
    switch (this.filter) {
      case 'active':
        return this.todos.filter(todo => !todo.completed);
      case 'completed':
        return this.todos.filter(todo => todo.completed);
      default:
        return this.todos;
    }
  }
  
  get activeCount(): number {
    return this.todos.filter(todo => !todo.completed).length;
  }
  
  async loadTodos(): Promise<void> {
    this.loading = true;
    this.error = null;
    
    try {
      this.todos = await this.todoService.fetchTodos();
    } catch (err) {
      this.error = 'Failed to load todos';
    } finally {
      this.loading = false;
    }
  }
  
  async addTodo(text: string, priority: Todo['priority']): Promise<void> {
    try {
      const newTodo = await this.todoService.createTodo({ text, completed: false, priority });
      this.todos.push(newTodo);
    } catch (err) {
      this.error = 'Failed to create todo';
    }
  }
  
  setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.filter = filter;
  }
}
```

```typescript
// View with MobX observer
import { observer } from 'mobx-react-lite';
import { useState, useEffect } from 'react';
import { TodoListViewModel } from './TodoListViewModel';

export const TodoListView = observer(() => {
  const [viewModel] = useState(() => new TodoListViewModel(new TodoService()));
  
  useEffect(() => {
    viewModel.loadTodos();
  }, [viewModel]);
  
  if (viewModel.loading) {
    return <div>Loading...</div>;
  }
  
  return (
    <div>
      {viewModel.filteredTodos.map(todo => (
        <div key={todo.id}>{todo.text}</div>
      ))}
    </div>
  );
});
```

### Using Custom Hooks (React-Specific Pattern)

An alternative approach uses custom hooks to encapsulate ViewModel logic:

```typescript
// Custom hook as ViewModel
export function useTodoListViewModel() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');
  
  const todoService = useMemo(() => new TodoService(), []);
  
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
  
  const loadTodos = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const data = await todoService.fetchTodos();
      setTodos(data);
    } catch (err) {
      setError('Failed to load todos');
    } finally {
      setLoading(false);
    }
  }, [todoService]);
  
  const addTodo = useCallback(async (text: string, priority: Todo['priority']) => {
    try {
      const newTodo = await todoService.createTodo({ text, completed: false, priority });
      setTodos(prev => [...prev, newTodo]);
    } catch (err) {
      setError('Failed to create todo');
    }
  }, [todoService]);
  
  return {
    // State
    filteredTodos,
    loading,
    error,
    filter,
    activeCount,
    
    // Commands
    loadTodos,
    addTodo,
    setFilter,
  };
}
```

```typescript
// View using custom hook
export function TodoListView() {
  const viewModel = useTodoListViewModel();
  
  useEffect(() => {
    viewModel.loadTodos();
  }, [viewModel.loadTodos]);
  
  return (
    <div>
      {viewModel.filteredTodos.map(todo => (
        <div key={todo.id}>{todo.text}</div>
      ))}
    </div>
  );
}
```

---

## Common Patterns

### 1. Computed Properties

Derived values that automatically update when dependencies change:

```typescript
export class TodoListViewModel {
  todos: Todo[] = [];
  
  get completionPercentage(): number {
    if (this.todos.length === 0) return 0;
    const completed = this.todos.filter(t => t.completed).length;
    return Math.round((completed / this.todos.length) * 100);
  }
  
  get hasHighPriorityTodos(): boolean {
    return this.todos.some(t => t.priority === 'high' && !t.completed);
  }
}
```

### 2. Command Pattern

Encapsulating user actions as methods:

```typescript
export class TodoListViewModel {
  async clearCompleted(): Promise<void> {
    const completedIds = this.todos
      .filter(t => t.completed)
      .map(t => t.id);
    
    for (const id of completedIds) {
      await this.todoService.deleteTodo(id);
    }
    
    this.todos = this.todos.filter(t => !t.completed);
  }
  
  async toggleAll(): Promise<void> {
    const allCompleted = this.todos.every(t => t.completed);
    const newStatus = !allCompleted;
    
    for (const todo of this.todos) {
      await this.todoService.updateTodo(todo.id, { completed: newStatus });
    }
    
    this.todos = this.todos.map(t => ({ ...t, completed: newStatus }));
  }
}
```

### 3. Validation

Centralizing validation logic in ViewModel:

```typescript
export class TodoFormViewModel {
  text: string = '';
  priority: Todo['priority'] = 'medium';
  validationErrors: string[] = [];
  
  get isValid(): boolean {
    return this.validationErrors.length === 0;
  }
  
  validateText(text: string): void {
    this.text = text;
    this.validationErrors = TodoValidator.validate({ text });
  }
  
  async submit(): Promise<boolean> {
    if (!this.isValid) {
      return false;
    }
    
    await this.todoService.createTodo({
      text: this.text,
      completed: false,
      priority: this.priority,
    });
    
    this.reset();
    return true;
  }
  
  reset(): void {
    this.text = '';
    this.priority = 'medium';
    this.validationErrors = [];
  }
}
```

### 4. Loading and Error States

Managing asynchronous operation states:

```typescript
export class TodoListViewModel {
  private operationState: 'idle' | 'loading' | 'error' | 'success' = 'idle';
  private errorMessage: string | null = null;
  
  get isLoading(): boolean {
    return this.operationState === 'loading';
  }
  
  get isError(): boolean {
    return this.operationState === 'error';
  }
  
  get error(): string | null {
    return this.errorMessage;
  }
  
  private async executeOperation<T>(operation: () => Promise<T>): Promise<T | null> {
    this.operationState = 'loading';
    this.errorMessage = null;
    
    try {
      const result = await operation();
      this.operationState = 'success';
      return result;
    } catch (err) {
      this.operationState = 'error';
      this.errorMessage = err instanceof Error ? err.message : 'Unknown error';
      return null;
    }
  }
}
```

---

## Anti-Patterns

### 1. View Logic in ViewModel

The ViewModel should not contain UI-specific logic:

```typescript
// Incorrect - ViewModel contains UI logic
export class TodoListViewModel {
  async deleteTodo(id: string): Promise<void> {
    const confirmed = window.confirm('Delete this todo?'); // UI logic in ViewModel
    if (!confirmed) return;
    
    await this.todoService.deleteTodo(id);
    this.todos = this.todos.filter(t => t.id !== id);
  }
}

// Correct - View handles UI logic
export function TodoListView() {
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
  const [todoService] = useState(() => new TodoService()); // Direct Model access
  
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
  }, [viewModel]);
  
  return <div>...</div>;
}
```

### 3. Business Logic in ViewModel

Complex business rules belong in the Model, not ViewModel:

```typescript
// Incorrect - Business logic in ViewModel
export class TodoListViewModel {
  calculatePriorityScore(todo: Todo): number {
    // Complex business calculation should be in Model
    let score = 0;
    if (todo.priority === 'high') score += 10;
    if (todo.priority === 'medium') score += 5;
    if (!todo.completed) score += 3;
    const daysSinceCreation = (Date.now() - todo.createdAt.getTime()) / (1000 * 60 * 60 * 24);
    score += daysSinceCreation * 0.1;
    return score;
  }
}

// Correct - Business logic in Model
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

export class TodoListViewModel {
  get sortedByPriority(): Todo[] {
    return [...this.todos].sort((a, b) => 
      TodoPriorityCalculator.calculateScore(b) - TodoPriorityCalculator.calculateScore(a)
    );
  }
}
```

### 4. God ViewModel

Avoid creating ViewModels that manage too many responsibilities:

```typescript
// Incorrect - Single ViewModel managing everything
export class ApplicationViewModel {
  todos: Todo[] = [];
  user: User | null = null;
  settings: Settings = {};
  notifications: Notification[] = [];
  // Too many concerns in one ViewModel
}

// Correct - Separate ViewModels for each concern
export class TodoListViewModel {
  todos: Todo[] = [];
  // Only todo-related logic
}

export class UserViewModel {
  user: User | null = null;
  // Only user-related logic
}

export class NotificationViewModel {
  notifications: Notification[] = [];
  // Only notification-related logic
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

---

## Why do we use MVVM?

- Presentation logic complexity justifies abstraction
- Forms require complex validation in addition to data manipulation
- Multiple views will share the same ViewModel
- Separation of concerns facilitates comprehensive unit testing
- Allows for parallel development

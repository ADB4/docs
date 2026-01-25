# TypeScript Conditionals and Loops: Reference Guide

# Table of Contents

- [Using a Todo List App as Example](#using-a-todo-list-app-as-example)
- [1: Control Flow](#part-1-control-flow)
  - [Conditionals](#conditionals)
    - [Switch Statements](#switch-statements)
    - [Ternary Operator](#ternary-operator)
  - [Iteration](#iteration)
    - [for...of Loop (Recommended Approach)](#forof-loop-recommended-approach)
    - [for...in Loop (For Objects)](#forin-loop-for-objects)
    - [Traditional for Loop (not "wrong", but less readable)](#traditional-for-loop-not-wrong-but-less-readable)
    - [while Loop](#while-loop)
- [2: TypeScript-Specific Features](#part-2-typescript-specific-features)
  - [Type Guards and Type Narrowing](#type-guards-and-type-narrowing)
  - [Discriminated Unions](#discriminated-unions)
  - [Optional Chaining and Nullish Coalescing](#optional-chaining-and-nullish-coalescing)
  - [Array Methods](#array-methods)
  - [Object Iteration Methods](#object-iteration-methods)
- [3: Best Practices](#part-3-best-practices)
  - [Use Early Returns to Reduce Nesting](#use-early-returns-to-reduce-nesting)
  - [Prefer for...of Over Traditional for Loop](#prefer-forof-over-traditional-for-loop)
  - [Use const for Loop Variables](#use-const-for-loop-variables)
  - [Type Your Variables](#type-your-variables)
  - [Use Enums for Constants](#use-enums-for-constants)
  - [Create Type Predicates for Reusable Type Guards](#create-type-predicates-for-reusable-type-guards)
  - [Exhaustiveness Checking](#exhaustiveness-checking)
- [4: Common Pitfalls and Caveats](#part-4-common-pitfalls-and-caveats)
  - [Truthy/Falsy Checks Can Be Misleading](#truthyfalsy-checks-can-be-misleading)
  - [Don't Use for...in for Arrays (use for...OF)](#dont-use-forin-for-arrays-use-forof)
  - [Async Operations in Loops](#async-operations-in-loops)
  - [Don't Modify Arrays While Iterating](#dont-modify-arrays-while-iterating)
  - [Switch Statement Fall-Through](#switch-statement-fall-through)
  - [Optional Chaining Doesn't Prevent All Errors](#optional-chaining-doesnt-prevent-all-errors)
  - [Creating Functions Inside Loops](#creating-functions-inside-loops)

## Using a Todo List App as Example

```typescript
// Types we'll use throughout
type Todo = {
  id: number;
  title: string;
  completed: boolean;
  priority: 'low' | 'medium' | 'high';
  dueDate?: Date;
};
```

## Part 1: Control Flow
### Conditionals

#### Switch Statements

```typescript
function getPriorityColor(priority: string): string {
  switch (priority) {
    case 'high':
      return 'red';
    case 'medium':
      return 'yellow';
    case 'low':
      return 'green';
    default:
      return 'gray';
  }
}
```

#### Ternary Operator

```typescript
const status = todo.completed ? 'Done' : 'Pending';

// Can be nested (but use sparingly)
const urgency = todo.priority === 'high' 
                ? 'Urgent' : todo.priority === 'medium' 
                ? 'Normal' : 'Can wait'; 
                // nesting possible, but not recommended
```

### Iteration

#### for...of Loop (Recommended Approach)

To iterate over an *array*

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' }
];

for (const todo of todos) {
  console.log(todo.title);
}
```

#### for...in Loop (For Objects)
While arrays are iterable, **objects are not.** To iterate over an objects keys, use a for...in loop.
```typescript
const todoStats = { total: 10, completed: 7, pending: 3 };

for (const key in todoStats) {
  console.log(`${key}: ${todoStats[key]}`);
}
```


#### Traditional for Loop (not "wrong", but less readable)

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' },
  { id: 3, title: 'Read book', completed: false, priority: 'low' }
];

for (let i = 0; i < todos.length; i++) {
  console.log(todos[i].title);
}
```

#### while Loop

```typescript
let index = 0;
while (index < todos.length && !todos[index].completed) {
  console.log(`Still pending: ${todos[index].title}`);
  index++;
}
```


## Part 2: TypeScript-Specific Features

### Type Guards and Type Narrowing

TypeScript can automatically narrow types based on conditionals:

```typescript
function displayTodoInfo(todo: Todo | null) {
  if (todo === null) {
    console.log('No todo found');
    return;
  }
  // TypeScript knows todo is not null here
  console.log(todo.title);
}

// typeof for primitive types
function processTodoId(id: string | number) {
  if (typeof id === 'string') {
    return parseInt(id); // TypeScript knows it's a string
  }
  return id; // TypeScript knows it's a number
}

// Check for optional properties
function checkDueDate(todo: Todo) {
  if (todo.dueDate) {
    // TypeScript knows dueDate exists here
    console.log(`Due: ${todo.dueDate.toLocaleDateString()}`);
  }
}
```

### Discriminated Unions

```typescript
type TodoAction = 
  | { type: 'add'; todo: Todo }
  | { type: 'toggle'; id: number }
  | { type: 'delete'; id: number }
  | { type: 'clear_completed' };

function handleTodoAction(action: TodoAction) {
  switch (action.type) {
    case 'add':
      return `Adding: ${action.todo.title}`; // TypeScript knows todo exists
    case 'toggle':
      return `Toggling todo ${action.id}`; // TypeScript knows id exists
    case 'delete':
      return `Deleting todo ${action.id}`;
    case 'clear_completed':
      return 'Clearing all completed todos';
  }
}
```

### Optional Chaining and Nullish Coalescing

```typescript
// Optional chaining (?.)
const dueDate = todo?.dueDate?.toLocaleDateString(); // Safe navigation

// Nullish coalescing (??)
const priority = todo.priority ?? 'low'; // Use 'low' if priority is null/undefined

// Combined
const dueDateText = todo?.dueDate?.toLocaleDateString() ?? 'No due date';
```

### Array Methods

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' },
  { id: 3, title: 'Read book', completed: false, priority: 'low' },
  { id: 4, title: 'Exercise', completed: true, priority: 'medium' }
];

// map - transform each todo
const todoTitles = todos.map(todo => todo.title);
// ['Buy groceries', 'Write code', 'Read book', 'Exercise']

// filter - get only incomplete todos
const pendingTodos = todos.filter(todo => !todo.completed);
// [{ id: 1, ... }, { id: 3, ... }]

// reduce - count completed todos
const completedCount = todos.reduce((count, todo) => 
  todo.completed ? count + 1 : count, 0
); // 2

// find - get a specific todo by id
const todo = todos.find(todo => todo.id === 2);
// { id: 2, title: 'Write code', ... }

// some - check if any todos are high priority
const hasHighPriority = todos.some(todo => todo.priority === 'high'); // true

// every - check if all todos are completed
const allCompleted = todos.every(todo => todo.completed); // false
```

### Object Iteration Methods

```typescript
const todoStats = { 
  total: 10, 
  completed: 7, 
  pending: 3,
  highPriority: 2
};

// Iterate over key-value pairs
for (const [key, value] of Object.entries(todoStats)) {
  console.log(`${key}: ${value}`);
}

// Just keys
const statKeys = Object.keys(todoStats); 
// ['total', 'completed', 'pending', 'highPriority']

// Just values
const statValues = Object.values(todoStats); 
// [10, 7, 3, 2]
```

## Part 3: Best Practices

### Use Early Returns to Reduce Nesting

```typescript
// Bad: Deeply nested
function getTodoPriority(todo: Todo | null): string {
  if (todo) {
    if (todo.completed) {
      return 'Completed';
    } else {
      if (todo.priority === 'high') {
        return 'Urgent';
      } else {
        return 'Normal';
      }
    }
  } else {
    return 'No todo';
  }
}

// Good: Early returns
function getTodoPriority(todo: Todo | null): string {
  if (!todo) return 'No todo';
  if (todo.completed) return 'Completed';
  if (todo.priority === 'high') return 'Urgent';
  return 'Normal';
}
```

### Prefer for...of Over Traditional for Loop

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' }
];

// Less readable
for (let i = 0; i < todos.length; i++) {
  console.log(todos[i].title);
}

// More readable
for (const todo of todos) {
  console.log(todo.title);
}
```

### Use const for Loop Variables

```typescript
// Prevents accidental reassignment
for (const todo of todos) {
  // todo cannot be reassigned
  console.log(todo.title);
}

// Only use let when reassignment is needed
for (let i = 0; i < todos.length; i++) {
  // i needs to be reassigned
}
```

### Type Your Variables

```typescript
// Explicit typing when not obvious
const pendingTodos: Todo[] = todos.filter(todo => !todo.completed);

// Type inference works in most cases
for (const [index, todo] of todos.entries()) {
  // index is number, todo is Todo
  console.log(`${index}: ${todo.title}`);
}
```

### Use Enums for Constants

```typescript
enum Priority {
  Low = 'low',
  Medium = 'medium',
  High = 'high'
}

function filterTodosByPriority(todos: Todo[], priority: Priority) {
  return todos.filter(todo => todo.priority === priority);
}
filterTodosByPriority(todos, Priority.High);
```

### Create Type Predicates for Reusable Type Guards

```typescript
// Reusable type guard function
function isHighPriority(todo: Todo): todo is Todo & { priority: 'high' } {
  return todo.priority === 'high';
}

function isOverdue(todo: Todo): boolean {
  if (!todo.dueDate) return false;
  return todo.dueDate < new Date();
}

// Use them
const urgentTodos = todos.filter(isHighPriority);
const overdueTodos = todos.filter(isOverdue);
```

### Exhaustiveness Checking

```typescript
type TodoAction = 
  | { type: 'add'; todo: Todo }
  | { type: 'toggle'; id: number }
  | { type: 'delete'; id: number };

function todoReducer(todos: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'add':
      return [...todos, action.todo];
    case 'toggle':
      return todos.map(todo => 
        todo.id === action.id 
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'delete':
      return todos.filter(todo => todo.id !== action.id);
    default:
      // Compiler error if we add a new Action type and forget to handle it
      const _exhaustive: never = action;
      return _exhaustive;
  }
}
```

## Part 4: Common Pitfalls and Caveats

### Truthy/Falsy Checks Can Be Misleading

```typescript
// PITFALL: 0 and empty strings are falsy but might be valid values
const todoId = 0; // Valid ID for first todo
if (todoId) {
  loadTodo(todoId); // Never runs! 0 is falsy
}

const todoTitle = ''; // Empty title might be allowed temporarily
if (todoTitle) {
  saveTodo(todoTitle); // Never runs for empty string
}

// Better: Be explicit
if (todoId !== undefined && todoId !== null) {
  loadTodo(todoId);
}

// Or use nullish check (only null/undefined are falsy)
if (todoId != null) {
  loadTodo(todoId);
}

// For strings, check explicitly for empty string if that matters
if (todoTitle !== undefined && todoTitle !== null && todoTitle !== '') {
  saveTodo(todoTitle);
}
```

### Don't Use for...in for Arrays (use for...OF)

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' }
];

// PITFALL: for...in iterates over keys, not values
// It can also include prototype properties
for (const index in todos) {
  console.log(index); // '0', '1' (strings, not numbers!)
  console.log(typeof index); // 'string'
  console.log(todos[index].title); // Works but not recommended
}

// Correct: Use for...of for values
for (const todo of todos) {
  console.log(todo.title); // Direct access to todo object
}

// for...in is for objects
const todoStats = { total: 10, completed: 7, pending: 3 };
for (const key in todoStats) {
  if (todoStats.hasOwnProperty(key)) { // Safety check
    console.log(key, todoStats[key]);
  }
}
```

### Async Operations in Loops

```typescript
async function saveTodo(todo: Todo): Promise<void> {
  // Simulated API call
  await new Promise(resolve => setTimeout(resolve, 100));
}

const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' },
  { id: 3, title: 'Exercise', completed: false, priority: 'low' }
];

// PITFALL: This runs serially (one at a time) - can be slow
async function saveAllTodosSerial() {
  for (const todo of todos) {
    await saveTodo(todo); // Each waits for the previous to complete
  }
  // Takes 300ms for 3 todos
}

// Better: Parallel execution when order doesn't matter
async function saveAllTodosParallel() {
  await Promise.all(todos.map(todo => saveTodo(todo)));
  // Takes 100ms for 3 todos (all run simultaneously)
}

// Note: Serial execution is correct when you need it
async function updateTodosInOrder() {
  for (const todo of todos) {
    await saveTodo(todo); // Intentional: each must complete before next
  }
}
```

### Don't Modify Arrays While Iterating

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' },
  { id: 3, title: 'Exercise', completed: true, priority: 'low' }
];

// PITFALL: Modifying array during iteration causes unexpected behavior
for (const todo of todos) {
  if (todo.completed) {
    // bad: This changes array length while iterating
    todos.splice(todos.indexOf(todo), 1); 
    // skips next element while iterating
  }
}

// instead: Use filter to create new array
const activeTodos = todos.filter(todo => !todo.completed);

// Or if you must modify in place, iterate backwards
for (let i = todos.length - 1; i >= 0; i--) {
  if (todos[i].completed) {
    todos.splice(i, 1);
  }
}
```

### Switch Statement Fall-Through

```typescript
function getTodoUrgency(priority: string): string {
  let urgency: string;
  
  // PITFALL: Forgetting break causes fall-through
  switch (priority) {
    case 'high':
      urgency = 'Urgent';
      // missing break
    case 'medium':
      urgency = 'Normal'; // this also runs for 'high'!
      break;
    case 'low':
      urgency = 'Low';
      break;
  }
  
  return urgency;
}

// instead: always use break (unless fall-through is intentional)
function getTodoUrgency(priority: string): string {
  switch (priority) {
    case 'high':
      return 'Urgent';
    case 'medium':
      return 'Normal';
    case 'low':
      return 'Low';
    default:
      return 'Unknown';
  }
}
```

### Optional Chaining Doesn't Prevent All Errors

```typescript
const todo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  priority: 'medium',
  dueDate: new Date()
};

// pitfall: Optional chaining stops at null/undefined, not other errors
const year = todo?.dueDate?.getFullYear(); // Works

// But if dueDate is a string by mistake, this throws
const invalidTodo: any = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  priority: 'medium',
  dueDate: '2024-01-01' // String instead of Date
};

const invalidYear = invalidTodo?.dueDate?.getFullYear(); 
// throws an error: String doesn't have getFullYear()

// It's not a replacement for try-catch
try {
  const formattedDate = todo.dueDate?.toLocaleDateString();
} catch (error) {
  console.error('Error formatting date:', error);
}
```

### Creating Functions Inside Loops

```typescript
const todos: Todo[] = [
  { id: 1, title: 'Buy groceries', completed: false, priority: 'medium' },
  { id: 2, title: 'Write code', completed: true, priority: 'high' }
];

// PITFALL: Creates new function on each iteration (performance impact)
const buttons: HTMLElement[] = [];
for (let i = 0; i < todos.length; i++) {
  buttons[i].addEventListener('click', function() {
    console.log(i); // Also closure issue! Always logs todos.length
    toggleTodo(todos[i]);
  });
}

// Better: Define function outside
function createToggleHandler(todo: Todo, index: number) {
  return () => {
    console.log(index);
    toggleTodo(todo);
  };
}

for (let i = 0; i < todos.length; i++) {
  buttons[i].addEventListener('click', createToggleHandler(todos[i], i));
}

// Or use forEach with arrow functions
todos.forEach((todo, index) => {
  buttons[index].addEventListener('click', () => {
    console.log(index); // Correctly captures index
    toggleTodo(todo);
  });
});
```


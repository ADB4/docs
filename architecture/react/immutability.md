# Pass by Reference/Value & Shallow/Deep Copies in React TypeScript

# Table of Contents

# Table of Contents

- [1: Pass by Value vs Pass by Reference](#1-pass-by-value-vs-pass-by-reference)
  - [Pass by Value (Primitives)](#pass-by-value-primitives)
  - [Pass by Reference (Objects)](#pass-by-reference-objects)
  - [Arrays are Objects (Pass by Reference)](#arrays-are-objects-pass-by-reference)
- [2: Shallow vs Deep Copies](#2-shallow-vs-deep-copies)
  - [Shallow Copy](#shallow-copy)
    - [Shallow Copy Methods](#shallow-copy-methods)
    - [Shallow Copy Behavior](#shallow-copy-behavior)
  - [Deep Copy](#deep-copy)
    - [Deep Copy Methods](#deep-copy-methods)
    - [Deep Copy Behavior](#deep-copy-behavior)
- [3: React State and Immutability](#3-react-state-and-immutability)
  - [Why Immutability Matters in React](#why-immutability-matters-in-react)
  - [Common React State Update Patterns](#common-react-state-update-patterns)
    - [Updating Arrays](#updating-arrays)
    - [Updating Nested Objects](#updating-nested-objects)
    - [Using Immer for Complex Updates](#using-immer-for-complex-updates)
- [4: Common Pitfalls](#4-common-pitfalls)
  - [Array Methods that Mutate](#array-methods-that-mutate)
  - [Shallow Copy Pitfall with Nested Data](#shallow-copy-pitfall-with-nested-data)
  - [Object.assign() Pitfall](#objectassign-pitfall)
  - [Destructuring Doesn't Deep Copy](#destructuring-doesnt-deep-copy)
  - [Setting State with Stale Values](#setting-state-with-stale-values)
- [5: Best Practices](#5-best-practices)
  - [Use Functional Updates for State Based on Previous State](#use-functional-updates-for-state-based-on-previous-state)
  - [Use Spread Operator for Shallow Copies](#use-spread-operator-for-shallow-copies)
  - [Use structuredClone for Deep Copies (When Needed)](#use-structuredclone-for-deep-copies-when-needed)
  - [Avoid Mutating Methods](#avoid-mutating-methods)
- [Quick Reference](#quick-reference)

## 1: Pass by Value vs Pass by Reference

In JavaScript/TypeScript, the way values are passed depends on the data type:

When we say **pass by value**, we mean a copy of an object and its values are passed(for example, into a function)

In **pass by reference**, a reference to the object is passed, meaning any modifications made to the object persist beyond the function scope.
#### Pass by Value (Primitives)

Primitives are copied when assigned or passed to functions:

```typescript
// primitives: string, number, boolean, null, undefined, symbol, bigint
let count = 5;
let newCount = count; // creates a COPY

newCount = 10;

console.log(count);    // 5 (unchanged)
console.log(newCount); // 10
```

```typescript
function incrementCount(num: number) {
  num = num + 1;
  return num;
}

let todoCount = 5;
const result = incrementCount(todoCount); // passes a copy

console.log(todoCount); // 5 (unchanged)
console.log(result);    // 6
```

#### Pass by Reference (Objects)

Objects, arrays, and functions are passed by reference (the variable holds a reference/pointer to the data):

```typescript
type Todo = {
  id: number;
  title: string;
  completed: boolean;
};

const todo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false
};

const sameTodo = todo; // copies the REFERENCE, not the object

sameTodo.completed = true;

console.log(todo.completed);     // true (changed)
console.log(sameTodo.completed); // true
console.log(todo === sameTodo);  // true (same reference)
```

```typescript
function markComplete(todo: Todo) {
  todo.completed = true; // modifies the original object
}

const myTodo: Todo = { id: 1, title: 'Write code', completed: false };
markComplete(myTodo); // passes reference to the object

console.log(myTodo.completed); // true (original was modified)
```

### Arrays are Objects (Pass by Reference)

```typescript
const todos = [
  { id: 1, title: 'Buy groceries', completed: false },
  { id: 2, title: 'Write code', completed: false }
];

const sameTodos = todos; // copies the REFERENCE

sameTodos[0].completed = true;

console.log(todos[0].completed); // true (original array modified)
console.log(todos === sameTodos); // true (same reference)
```

## 2: Shallow vs Deep Copies

### Shallow Copy

A shallow copy creates a new object/array, but only copies the first level. Nested objects/arrays are still referenced:

#### Shallow Copy Methods

```typescript
type Todo = {
  id: number;
  title: string;
  completed: boolean;
  tags?: string[];
};

const originalTodo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  tags: ['shopping', 'food']
};

// Method 1: Spread operator (most common)
const copy1 = { ...originalTodo };

// Method 2: Object.assign()
const copy2 = Object.assign({}, originalTodo);

// Method 3: Array spread for arrays
const todos = [originalTodo];
const todosCopy = [...todos];
```

#### Shallow Copy Behavior

```typescript
const originalTodo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  tags: ['shopping', 'food']
};

const shallowCopy = { ...originalTodo };

// Top-level properties are independent
shallowCopy.completed = true;
console.log(originalTodo.completed); // false (unchanged)

// BUT nested objects/arrays are still referenced
shallowCopy.tags?.push('urgent');
console.log(originalTodo.tags); // ['shopping', 'food', 'urgent'] (changed)

console.log(originalTodo === shallowCopy);           // false (different objects)
console.log(originalTodo.tags === shallowCopy.tags); // true (same array reference)
```

### Deep Copy

A deep copy creates a completely independent copy, including all nested objects/arrays:

#### Deep Copy Methods

```typescript
type Todo = {
  id: number;
  title: string;
  completed: boolean;
  tags: string[];
  metadata?: {
    createdAt: Date;
    priority: 'low' | 'medium' | 'high';
  };
};

const originalTodo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  tags: ['shopping', 'food'],
  metadata: {
    createdAt: new Date(),
    priority: 'high'
  }
};

// Method 1: JSON parse/stringify (most common, but has limitations)
const deepCopy1 = JSON.parse(JSON.stringify(originalTodo));
// loses functions, Date objects become strings, undefined becomes null

// Method 2: structuredClone (modern browsers, Node 17+)
const deepCopy2 = structuredClone(originalTodo);
// ✓ Handles Date objects, Maps, Sets, etc.

// Method 3: manual recursive copy
function deepClone<T>(obj: T): T {
  if (obj === null || typeof obj !== 'object') return obj;
  
  if (obj instanceof Date) return new Date(obj.getTime()) as T;
  if (obj instanceof Array) return obj.map(item => deepClone(item)) as T;
  
  const cloned = {} as T;
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}

const deepCopy3 = deepClone(originalTodo);
```

#### Deep Copy Behavior

```typescript
const originalTodo: Todo = {
  id: 1,
  title: 'Buy groceries',
  completed: false,
  tags: ['shopping', 'food'],
  metadata: {
    createdAt: new Date(),
    priority: 'high'
  }
};

const deepCopy = structuredClone(originalTodo);

// top-level changes don't affect original
deepCopy.completed = true;
console.log(originalTodo.completed); // false

// tested changes ALSO don't affect original
deepCopy.tags.push('urgent');
console.log(originalTodo.tags); // ['shopping', 'food'] (unchanged)

deepCopy.metadata!.priority = 'low';
console.log(originalTodo.metadata!.priority); // 'high' (unchanged)

console.log(originalTodo === deepCopy);                     // false
console.log(originalTodo.tags === deepCopy.tags);           // false
console.log(originalTodo.metadata === deepCopy.metadata);   // false
```

## 3: React State and Immutability

### Why Immutability Matters in React

React detects state changes by comparing references. If you mutate an object directly, React won't detect the change:

```typescript
import { useState } from 'react';

type Todo = {
  id: number;
  title: string;
  completed: boolean;
};

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, title: 'Buy groceries', completed: false }
  ]);

  // WRONG: mutating state directly
  const handleToggleWrong = (id: number) => {
    const todo = todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed; // mutates the original
      setTodos(todos); // react won't re-render. Same reference
    }
  };

  // CORRECT: create new array with updated object
  const handleToggleCorrect = (id: number) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed } // new object
        : todo
    )); // new array
  };

  return (
    <div>
      {todos.map(todo => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => handleToggleCorrect(todo.id)}
          />
          {todo.title}
        </div>
      ))}
    </div>
  );
}
```

### Common React State Update Patterns

#### Updating Arrays

```typescript
const [todos, setTodos] = useState<Todo[]>([]);

// add item
const addTodo = (newTodo: Todo) => {
  setTodos([...todos, newTodo]); // new array
  // OR
  setTodos(prev => [...prev, newTodo]);
};

// remove item
const removeTodo = (id: number) => {
  setTodos(todos.filter(todo => todo.id !== id)); // new array
};

// update item
const updateTodo = (id: number, updates: Partial<Todo>) => {
  setTodos(todos.map(todo =>
    todo.id === id
      ? { ...todo, ...updates } // new object with updates
      : todo
  ));
};

// replace entire array
const clearCompleted = () => {
  setTodos(todos.filter(todo => !todo.completed));
};
```

#### Updating Nested Objects

```typescript
type TodoWithMetadata = {
  id: number;
  title: string;
  completed: boolean;
  metadata: {
    createdAt: Date;
    priority: 'low' | 'medium' | 'high';
    tags: string[];
  };
};

const [todo, setTodo] = useState<TodoWithMetadata>({
  id: 1,
  title: 'Buy groceries',
  completed: false,
  metadata: {
    createdAt: new Date(),
    priority: 'high',
    tags: ['shopping']
  }
});

// BAD: nested mutation
const updatePriorityWrong = (priority: 'low' | 'medium' | 'high') => {
  todo.metadata.priority = priority; // Mutation
  setTodo(todo); // Won't trigger re-render
};

// BETTER: create new nested structure
const updatePriorityCorrect = (priority: 'low' | 'medium' | 'high') => {
  setTodo({
    ...todo,
    metadata: {
      ...todo.metadata,
      priority
    }
  });
};

// add tag to nested array
const addTag = (tag: string) => {
  setTodo({
    ...todo,
    metadata: {
      ...todo.metadata,
      tags: [...todo.metadata.tags, tag] // new array
    }
  });
};
```

#### Using Immer for Complex Updates

For deeply nested state, consider using Immer:

```typescript
import { useState } from 'react';
import { useImmer } from 'use-immer';

type ComplexTodo = {
  id: number;
  title: string;
  subtasks: {
    id: number;
    title: string;
    completed: boolean;
  }[];
};

// traditional approach (verbose)
function TodoWithUseState() {
  const [todo, setTodo] = useState<ComplexTodo>({
    id: 1,
    title: 'Main task',
    subtasks: []
  });

  const toggleSubtask = (subtaskId: number) => {
    setTodo({
      ...todo,
      subtasks: todo.subtasks.map(subtask =>
        subtask.id === subtaskId
          ? { ...subtask, completed: !subtask.completed }
          : subtask
      )
    });
  };
}

// with Immer (cleaner)
function TodoWithImmer() {
  const [todo, setTodo] = useImmer<ComplexTodo>({
    id: 1,
    title: 'Main task',
    subtasks: []
  });

  const toggleSubtask = (subtaskId: number) => {
    setTodo(draft => {
      const subtask = draft.subtasks.find(s => s.id === subtaskId);
      if (subtask) {
        subtask.completed = !subtask.completed; // looks like mutation, but Immer handles immutability
      }
    });
  };
}
```

## 4: Common Pitfalls

### Array Methods that Mutate

```typescript
const [todos, setTodos] = useState<Todo[]>([
  { id: 1, title: 'Buy groceries', completed: false }
]);

// these mutate the original array:
todos.push(newTodo);
todos.pop();
todos.shift();            // removes from start
todos.unshift(newTodo);   // adds to start
todos.splice(0, 1);       // removes/adds items
todos.sort();             // sorts in place
todos.reverse();          // reverses in place

setTodos(todos); // won't trigger re-render Same reference

// use these instead (return new arrays):
setTodos([...todos, newTodo]);           // instead of push
setTodos(todos.slice(0, -1));            // instead of pop
setTodos(todos.slice(1));                // instead of shift
setTodos([newTodo, ...todos]);           // instead of unshift
setTodos([...todos.slice(0, i), ...todos.slice(i + 1)]); // instead of splice
setTodos([...todos].sort());             // instead of sort
setTodos([...todos].reverse());          // instead of reverse
```

### Shallow Copy Pitfall with Nested Data

```typescript
type Todo = {
  id: number;
  title: string;
  tags: string[];
};

const [todo, setTodo] = useState<Todo>({
  id: 1,
  title: 'Buy groceries',
  tags: ['shopping', 'food']
});

// BAD: shallow copy doesn't copy nested arrays
const addTagWrong = (tag: string) => {
  const newTodo = { ...todo }; // shallow copy
  newTodo.tags.push(tag);      // mutates original tags array
  setTodo(newTodo);            // may or may not trigger re-render
};

// CORRECT: copy nested arrays too
const addTagCorrect = (tag: string) => {
  setTodo({
    ...todo,
    tags: [...todo.tags, tag] // new array
  });
};
```

### Object.assign() Pitfall

```typescript
type Todo = {
  id: number;
  title: string;
  metadata: {
    priority: 'low' | 'medium' | 'high';
  };
};

const [todo, setTodo] = useState<Todo>({
  id: 1,
  title: 'Buy groceries',
  metadata: { priority: 'high' }
});

// BAD: Object.assign does shallow copy
const updateWrong = () => {
  const newTodo = Object.assign({}, todo);
  newTodo.metadata.priority = 'low'; // mutates original!
  setTodo(newTodo);
};

// CORRECT: spread nested objects
const updateCorrect = () => {
  setTodo({
    ...todo,
    metadata: {
      ...todo.metadata,
      priority: 'low'
    }
  });
};
```

### Destructuring Doesn't Deep Copy

```typescript
type Todo = {
  id: number;
  title: string;
  metadata: {
    tags: string[];
  };
};

const originalTodo: Todo = {
  id: 1,
  title: 'Buy groceries',
  metadata: {
    tags: ['shopping']
  }
};

// destructuring is still shallow
const { ...copyTodo } = originalTodo;
copyTodo.metadata.tags.push('food'); // mutates original

console.log(originalTodo.metadata.tags); // ['shopping', 'food'] (modified)

// must manually copy nested structures
const properCopy = {
  ...originalTodo,
  metadata: {
    ...originalTodo.metadata,
    tags: [...originalTodo.metadata.tags]
  }
};
```

### Setting State with Stale Values

```typescript
function TodoCounter() {
  const [count, setCount] = useState(0);

  // problematic: using stale value in rapid updates
  const incrementThreeTimesWrong = () => {
    setCount(count + 1); // uses count = 0
    setCount(count + 1); // uses count = 0
    setCount(count + 1); // uses count = 0
    // final count: 1 (not 3)
  };

  // instead try: use functional updates
  const incrementThreeTimesCorrect = () => {
    setCount(prev => prev + 1); // uses latest value
    setCount(prev => prev + 1); // uses latest value
    setCount(prev => prev + 1); // uses latest value
    // final count: 3
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThreeTimesCorrect}>+3</button>
    </div>
  );
}
```

## 5: Best Practices

### Use Functional Updates for State Based on Previous State

```typescript
const [todos, setTodos] = useState<Todo[]>([]);

// better: use functional update when new state depends on old state
const addTodo = (newTodo: Todo) => {
  setTodos(prevTodos => [...prevTodos, newTodo]);
};

const toggleTodo = (id: number) => {
  setTodos(prevTodos =>
    prevTodos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    )
  );
};
```

### Use Spread Operator for Shallow Copies

```typescript
// arrays
const newTodos = [...todos, newTodo];
const filteredTodos = todos.filter(predicate);

// objects
const updatedTodo = { ...todo, completed: true };
const mergedTodo = { ...defaultTodo, ...customTodo };
```

### Use structuredClone for Deep Copies (When Needed)

```typescript
// when you need a true deep copy
const deepCopy = structuredClone(complexNestedObject);

```

### Avoid Mutating Methods

```typescript
// problematic
todos.push(newTodo);
todos.sort();
todo.tags.push('new');

// instead try:
const newTodos = [...todos, newTodo];
const sortedTodos = [...todos].sort();
const newTags = [...todo.tags, 'new'];
```



## Quick Reference

| operation | mutates Original? | use case |
|-----------|------------------|----------|
| `{ ...obj }` | no | shallow copy object |
| `[...arr]` | no | shallow copy array |
| `Object.assign()` | no | shallow copy object |
| `arr.map/filter` | no | transform array |
| `structuredClone()` | no | deep copy |
| `JSON.parse(JSON.stringify())` | no | deep copy (limited) |
| `arr.push/pop/shift` | yes | avoid in React |
| `arr.sort/reverse` | yes | avoid in React |
| `obj.prop = value` | yes | avoid in React |
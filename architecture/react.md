# React/TypeScript Component Declaration Best Practices

## Table of Contents
- [Component Declaration Patterns](#component-declaration-patterns)
- [Typing Props](#typing-props)
- [Named vs Default Exports](#named-vs-default-exports)
- [Best Practices](#best-practices)
- [Common Pitfalls](#common-pitfalls)
- [Reference Implementation](#reference-implementation)

---

## Type Definitions Used in Examples

Throughout this guide, examples reference a `Todo` type defined as:

```typescript
interface Todo {
  id: string;
  text: string;
  completed: boolean;
  priority?: 'low' | 'medium' | 'high';
}
```

---

## Component Declaration Patterns

### 1. Function Declaration

```typescript
export function TodoList({ todos, onToggle, onDelete }: TodoListProps) {
  return <div>...</div>;
}
```

**Advantages:**
- Function hoisting allows the component to be referenced before its declaration within the same file
- Function name appears in stack traces and React DevTools, improving debuggability
- Syntax is concise and does not require explicit type annotation on the function itself
- Aligns with standard JavaScript function declaration patterns

**Limitations:**
- Cannot be reassigned, though reassignment is rarely required and generally discouraged in component definitions

**Recommended use cases:**
- Standard approach for most component declarations
- Situations where forward references within a module are beneficial

---

### 2. Arrow Function with Type Declaration

```typescript
export const TodoList = ({ todos, onToggle, onDelete }: TodoListProps) => {
  return <div>...</div>;
};
```

**Advantages:**
- Uses arrow function syntax consistent with modern JavaScript patterns
- Can be incorporated into const assertions where required
- May align with team conventions for function expression usage

**Limitations:**
- Not hoisted; must be declared before any references in the same file
- May appear as anonymous in certain stack traces, though modern JavaScript engines typically infer the name from the variable assignment
- Requires additional syntax compared to function declarations

**Recommended use cases:**
- Enforcing immutable bindings through const declarations
- Maintaining consistency in codebases with established arrow function conventions
- Aligning with other arrow function exports in the same module

---

## Typing Props

### Interface vs Type Alias

Both interfaces and type aliases are viable for defining component props, with subtle differences in behavior and compiler performance.

#### Using Interface

```typescript
interface TodoListProps {
  todos: Todo[];
  onToggle: (id: string) => void;
  onDelete?: (id: string) => void;
}

export function TodoList({ todos, onToggle, onDelete }: TodoListProps) {
  return <div>...</div>;
}
```

**Characteristics:**
- Produces more readable error messages in TypeScript compiler output
- Supports declaration merging, allowing the same interface to be extended across multiple declarations
- Demonstrates marginally better performance in TypeScript compiler processing

#### Using Type Alias

```typescript
type TodoListProps = {
  todos: Todo[];
  onToggle: (id: string) => void;
  onDelete?: (id: string) => void;
};

export function TodoList({ todos, onToggle, onDelete }: TodoListProps) {
  return <div>...</div>;
}
```

**Characteristics:**
- Provides greater flexibility for complex type operations including unions, intersections, and mapped types
- Required for advanced type manipulations and conditional types

**General guidance:** Prefer `interface` for React component props in standard cases. Use `type` when advanced type features are necessary.

---

### Inline Props

```typescript
export function TodoList({ 
  todos,
  onToggle 
}: { 
  todos: Todo[];
  onToggle: (id: string) => void;
}) {
  return <div>...</div>;
}
```

This pattern has significant limitations:
- Props type cannot be exported or reused
- Reduces code maintainability for components that evolve over time

**Acceptable use cases:**
- Single-use components with no reusability requirements
- Extremely simple components with minimal props (1-2 properties)

---

## Named vs Default Exports

### Named Exports

```typescript
// TodoList.tsx
export function TodoList({ todos, onToggle }: TodoListProps) {
  return <div>...</div>;
}

// Usage
import { TodoList } from './TodoList';
```

**Advantages:**
- Improved IDE support for autocomplete and automated refactoring
- Prevents naming inconsistencies across different modules
- Allows multiple exports from a single module
- Generally provides better tree-shaking optimization in build tools

**Limitations:**
- Import statements require explicit braces syntax

---

### Default Exports

```typescript
// TodoList.tsx
export default function TodoList({ todos, onToggle }: TodoListProps) {
  return <div>...</div>;
}

// Usage
import TodoList from './TodoList';
import AlternativeName from './TodoList';  // Also valid, potentially problematic
```

**Advantages:**
- Slightly more concise import syntax
- Required for certain React APIs such as React.lazy() for code splitting

**Limitations:**
- Allows arbitrary naming at import sites, leading to potential inconsistency
- Complicates automated refactoring tools
- Reduces IDE autocomplete effectiveness

**Appropriate use cases:**
- Code splitting with React.lazy(): `const Component = lazy(() => import('./Component'))`
- Framework-specific requirements (e.g., Next.js page components, Remix route modules)

---

## Best Practices

### 1. Naming Conventions

Component names should follow PascalCase convention. Props interfaces should match the component name with a "Props" suffix.

```typescript
// Correct implementation
export function UserProfile({ userId }: UserProfileProps) {
  return <div>...</div>;
}

interface UserProfileProps {
  userId: string;
}

// Incorrect implementation
export function userProfile({ userId }: Props) {
  return <div>...</div>;
}
```

---

### 2. Props Destructuring

Destructuring props in function parameters improves readability and reduces repetitive object access.

```typescript
// Preferred approach
export function UserProfile({ userId, name, onUpdate }: UserProfileProps) {
  return <div>{name}</div>;
}

// Discouraged approach - requires repeated props object access
export function UserProfile(props: UserProfileProps) {
  return <div>{props.name}</div>;
}

// Acceptable when the full props object is needed
export function UserProfile(props: UserProfileProps) {
  const { userId, name } = props;
  
  useEffect(() => {
    trackView(props); // Passing entire props object to external function
  }, [props]);
  
  return <div>{name}</div>;
}
```

---

### 3. Default Props

Modern React with TypeScript uses default parameter syntax rather than the legacy defaultProps pattern.

```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

// Current standard approach
export function Button({ variant = 'primary', disabled = false }: ButtonProps) {
  return <button disabled={disabled}>{variant}</button>;
}

// Deprecated pattern - avoid in new code
Button.defaultProps = {
  variant: 'primary',
  disabled: false,
};
```

---

### 4. Children Prop

TypeScript requires explicit typing for the children prop. Two common patterns exist:

```typescript
// Explicit children in props interface
interface CardProps {
  title: string;
  children: React.ReactNode;
}

export function Card({ title, children }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Using PropsWithChildren utility type
import { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
}

export function Card({ title, children }: PropsWithChildren<CardProps>) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

---

### 5. Event Handlers

Event handler props should use descriptive names and include explicit types for event parameters.

```typescript
interface AddTodoFormProps {
  onSubmit: (text: string) => void;
  onChange?: (value: string) => void;
}

export function AddTodoForm({ onSubmit, onChange }: AddTodoFormProps) {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Process form data
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

---

## Common Pitfalls

### Unsafe Access to Optional Props

Optional props require defensive handling to prevent runtime errors.

```typescript
interface UserProps {
  user?: {
    name: string;
    email: string;
  };
}

// Problematic - throws error if user is undefined
export function UserDisplay({ user }: UserProps) {
  return <div>{user.name}</div>;
}

// Correct - uses optional chaining and nullish coalescing
export function UserDisplay({ user }: UserProps) {
  return <div>{user?.name ?? 'Guest'}</div>;
}

// Alternative - conditional rendering
export function UserDisplay({ user }: UserProps) {
  if (!user) {
    return <div>Guest</div>;
  }
  return <div>{user.name}</div>;
}
```

---

## Reference Implementation

### Standard Component Pattern

The following pattern represents the current recommended approach for React/TypeScript component declarations:

```typescript
// 1. Define and export props interface
export interface TodoItemProps {
  todo: Todo;
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
  showPriority?: boolean;
}

// 2. Use function declaration with named export
export function TodoItem({ 
  todo,
  onToggle,
  onDelete,
  showPriority = false
}: TodoItemProps) {
  // Component logic here
  
  return (
    <div>
      {/* JSX content */}
    </div>
  );
}
```
This pattern provides type safety, extensibility, maintainability, and is in alignment with current React team recommendations.

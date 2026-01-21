# React Hook Form: Critical Concepts Guide

A practical guide to understanding React Hook Form's core concepts, TypeScript integration, and schema validation.

---

## Table of Contents

1. [What is React Hook Form?](#what-is-react-hook-form)
2. [The useForm Hook](#the-useform-hook)
3. [TypeScript Integration](#typescript-integration)
4. [Validation Modes](#validation-modes)
5. [Schema Validation with Resolvers](#schema-validation-with-resolvers)
6. [Registering Fields](#registering-fields)
7. [Handling Submissions](#handling-submissions)
8. [Common Patterns and Gotchas](#common-patterns-and-gotchas)

---

## What is React Hook Form?

React Hook Form is a lightweight library for managing form state and validation in React applications. It minimizes re-renders and provides a simple API built around React hooks.

Core benefits include minimal re-renders through isolated component updates, small bundle size, straightforward integration with UI libraries, built-in TypeScript support, and flexible validation options (native or schema-based).

---

## The useForm Hook

The `useForm` hook is the foundation of React Hook Form. It returns methods and state for managing your form.

### Basic Usage

```typescript
const {
  control,        // For controlled components
  register,       // For registering uncontrolled inputs
  handleSubmit,   // Wraps your submit handler with validation
  reset,          // Resets form to default values
  watch,          // Subscribes to field changes
  formState,      // Contains errors, isDirty, isValid, etc.
  setValue,       // Programmatically set field values
  getValues,      // Get current form values
} = useForm({
  defaultValues: { ... },
  mode: 'onBlur',
  reValidateMode: 'onChange',
  resolver: yupResolver(schema),
});
```

### Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `defaultValues` | `object` | Initial values for all form fields |
| `mode` | `string` | When to validate before first submit |
| `reValidateMode` | `string` | When to validate after first submit |
| `resolver` | `function` | External validation (Yup, Zod, etc.) |
| `criteriaMode` | `string` | `'firstError'` or `'all'` errors per field |
| `shouldFocusError` | `boolean` | Focus first field with error on submit |

---

## TypeScript Integration

### Defining Your Form Type

Create a type that describes your form's data structure:

```typescript
export type LoginForm = {
  email: string;
  password: string;
  rememberMe: boolean;
};
```

### Applying the Type

Pass the type as a generic parameter to `useForm`:

```typescript
const { control, handleSubmit } = useForm<LoginForm>({
  defaultValues: {
    email: '',
    password: '',
    rememberMe: false,
  },
});
```

### What This Gives You

TypeScript now enforces correctness throughout your form:

```typescript
// In your submit handler
const onSubmit = (data: LoginForm) => {
  data.email       // TypeScript knows this is a string
  data.rememberMe  // TypeScript knows this is a boolean
  data.username    // Error: property doesn't exist
};

// When registering fields
<Controller
  name="email"      // Autocomplete suggests valid field names
  name="emails"     // Error: not a valid field name
  control={control}
  ...
/>
```

---

## Validation Modes

React Hook Form has two timing settings for validation: `mode` and `reValidateMode`.

### `mode` — Initial Validation Trigger

Controls when validation first runs, before the form has been submitted:

| Value | When Validation Runs |
|-------|---------------------|
| `'onSubmit'` | Only when user clicks submit (default) |
| `'onBlur'` | When a field loses focus |
| `'onChange'` | On every keystroke/change |
| `'onTouched'` | On first blur, then on every change |
| `'all'` | On both blur and change events |

Example scenarios:

```typescript
// User types in email field...

mode: 'onSubmit'   // No validation yet, wait for submit
mode: 'onBlur'     // No validation yet, wait for blur
mode: 'onChange'   // Validates immediately as they type
mode: 'all'        // Validates immediately as they type
```

### `reValidateMode` — Subsequent Validation Trigger

Controls when validation runs after the first submission (when errors are already displayed):

| Value | When Re-validation Runs |
|-------|------------------------|
| `'onSubmit'` | Only on next submit attempt |
| `'onBlur'` | When fields lose focus |
| `'onChange'` | On every keystroke (default) |

Why this matters:

```typescript
// Form was submitted with errors, user is now fixing them...

reValidateMode: 'onSubmit'
// Errors stay visible until next submit
// User doesn't know if they've fixed the issue

reValidateMode: 'onChange'
// Errors clear as soon as input becomes valid
// Immediate feedback that the fix worked
```

### Common Combinations

```typescript
// Lenient initially, responsive after errors
useForm({
  mode: 'onSubmit',
  reValidateMode: 'onChange',
});

// Immediate feedback always
useForm({
  mode: 'all',
  reValidateMode: 'onChange',
});

// Minimal validation (only on submit)
useForm({
  mode: 'onSubmit',
  reValidateMode: 'onSubmit',
});
```

---

## Schema Validation with Resolvers

### What is a Resolver?

React Hook Form doesn't natively understand external validation libraries like Yup or Zod. A resolver is an adapter that bridges the gap:

```
User Input --> React Hook Form --> Resolver --> Yup/Zod --> Validation Result --> React Hook Form
```

### Setting Up Yup Validation

1. Install dependencies:

```bash
npm install yup @hookform/resolvers
```

2. Create your schema:

```typescript
import * as yup from 'yup';

const loginSchema = yup.object({
  email: yup
    .string()
    .email('Please enter a valid email')
    .required('Email is required'),
  password: yup
    .string()
    .min(8, 'Password must be at least 8 characters')
    .required('Password is required'),
  rememberMe: yup.boolean(),
});
```

3. Connect with the resolver:

```typescript
import { yupResolver } from '@hookform/resolvers/yup';

const { control, handleSubmit } = useForm<LoginForm>({
  resolver: yupResolver(loginSchema),
  defaultValues: {
    email: '',
    password: '',
    rememberMe: false,
  },
});
```

### How the Resolver Works Internally

The `yupResolver` function returns a validation function that React Hook Form calls:

```typescript
// Simplified view of what yupResolver does
const yupResolver = (schema) => {
  return async (formData) => {
    try {
      const values = await schema.validate(formData, { abortEarly: false });
      return { values, errors: {} };
    } catch (validationErrors) {
      return {
        values: {},
        errors: formatYupErrors(validationErrors),
      };
    }
  };
};
```

### TypeScript and Schema Alignment

Your TypeScript type and Yup schema should match. If they don't, you'll get type errors:

```typescript
type LoginForm = {
  email: string;
  password: string;
  rememberMe: boolean;
};

// Problem: Schema missing 'rememberMe' causes TypeScript error
const schema = yup.object({
  email: yup.string().required(),
  password: yup.string().required(),
});

// Correct: Schema matches type exactly
const schema = yup.object({
  email: yup.string().required(),
  password: yup.string().required(),
  rememberMe: yup.boolean(),
});
```

### Conditional Schemas Gotcha

If you try to switch between schemas of different shapes, TypeScript will complain:

```typescript
const simpleSchema = yup.object({
  name: yup.string().required(),
});

const fullSchema = yup.object({
  name: yup.string().required(),
  email: yup.string().required(),
  phone: yup.string().required(),
});

// TypeScript error: schemas have different shapes
resolver: yupResolver(isFullForm ? fullSchema : simpleSchema)
```

Solutions:

1. Type assertion (quick fix):
```typescript
resolver: yupResolver(isFullForm ? fullSchema : simpleSchema) as any
```

2. Unified schema with conditional rules (type-safe):
```typescript
const schema = yup.object({
  name: yup.string().required(),
  email: isFullForm ? yup.string().required() : yup.string(),
  phone: isFullForm ? yup.string().required() : yup.string(),
});
```

---

## Registering Fields

### Uncontrolled: `register`

For native HTML inputs, use `register` for best performance:

```typescript
const { register } = useForm<LoginForm>();

<input {...register('email')} type="email" />
<input {...register('password')} type="password" />
```

### Controlled: `Controller`

For UI library components (Material UI, Ant Design, etc.), use `Controller`:

```typescript
import { Controller } from 'react-hook-form';

<Controller
  name="email"
  control={control}
  render={({ field, fieldState }) => (
    <TextField
      {...field}
      error={!!fieldState.error}
      helperText={fieldState.error?.message}
    />
  )}
/>
```

### When to Use Which

| Approach | Use Case |
|----------|----------|
| `register` | Native `<input>`, `<select>`, `<textarea>` |
| `Controller` | Third-party UI components, custom inputs |

---

## Handling Submissions

### Basic Submit Handler

```typescript
const onSubmit = (data: LoginForm) => {
  console.log('Valid data:', data);
  // Send to API, update state, etc.
};

const onError = (errors: FieldErrors<LoginForm>) => {
  console.log('Validation errors:', errors);
};

<form onSubmit={handleSubmit(onSubmit, onError)}>
  {/* fields */}
  <button type="submit">Submit</button>
</form>
```

### What `handleSubmit` Does

1. Prevents default form submission
2. Runs validation via the resolver
3. If valid: calls your `onSubmit` with clean data
4. If invalid: calls `onError` (optional) and updates `formState.errors`

### Accessing Form State

```typescript
const { formState: { errors, isSubmitting, isValid, isDirty } } = useForm();

// Display field errors
{errors.email && <span>{errors.email.message}</span>}

// Disable submit while processing
<button disabled={isSubmitting}>Submit</button>

// Show unsaved changes warning
{isDirty && <span>You have unsaved changes</span>}
```

---

## Common Patterns and Gotchas

### 1. Always Set Default Values

Without default values, fields start as `undefined`, which can cause issues:

```typescript
// Potential issues with undefined
useForm<LoginForm>();

// Preferred: Explicit defaults
useForm<LoginForm>({
  defaultValues: {
    email: '',
    password: '',
    rememberMe: false,
  },
});
```

### 2. Reset Form After Successful Submit

```typescript
const onSubmit = async (data: LoginForm) => {
  await api.login(data);
  reset(); // Clear form after success
};
```

### 3. Watch Field Values

```typescript
// Watch a single field
const email = watch('email');

// Watch multiple fields
const [email, password] = watch(['email', 'password']);

// Watch all fields (use sparingly, causes re-renders)
const allValues = watch();
```

### 4. Programmatically Set Values

```typescript
// Set a single field
setValue('email', 'user@example.com');

// Set and trigger validation
setValue('email', 'user@example.com', { shouldValidate: true });

// Set multiple fields
reset({ email: 'user@example.com', password: '', rememberMe: true });
```

### 5. DevTools for Debugging

```bash
npm install -D @hookform/devtools
```

```typescript
import { DevTool } from '@hookform/devtools';

<form>
  {/* fields */}
</form>
<DevTool control={control} />
```

---

## Quick Reference

### useForm Configuration

```typescript
useForm<FormType>({
  defaultValues: {},           // Initial form values
  mode: 'onSubmit',            // Validation timing (initial)
  reValidateMode: 'onChange',  // Validation timing (after submit)
  resolver: yupResolver(schema), // External validation
  criteriaMode: 'firstError',  // Show first or all errors
  shouldFocusError: true,      // Auto-focus first error
});
```

### Returned Methods

| Method | Purpose |
|--------|---------|
| `register` | Register uncontrolled inputs |
| `control` | For `Controller` components |
| `handleSubmit` | Wrap submit handler with validation |
| `watch` | Subscribe to field changes |
| `reset` | Reset form to defaults |
| `setValue` | Set field value programmatically |
| `getValues` | Get current values |
| `trigger` | Manually trigger validation |
| `formState` | Access errors, isDirty, isValid, etc. |

---

## Further Reading

- [React Hook Form Documentation](https://react-hook-form.com/)
- [Yup Schema Validation](https://github.com/jquense/yup)
- [@hookform/resolvers](https://github.com/react-hook-form/resolvers)
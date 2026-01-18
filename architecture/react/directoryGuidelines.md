# React/TypeScript/MUI Frontend Structure Guidelines

A comprehensive guide for organizing production-grade React applications using TypeScript and Material-UI with a page-based architecture.

## Project Structure

```
src/
├── assets/              # Static files (images, fonts, icons)
├── components/          # Reusable UI components
│   ├── common/         # Shared components (Button, Card, Modal)
│   ├── forms/          # Form-specific components
│   └── layout/         # Layout components (Header, Footer, Sidebar)
├── pages/              # Page components (route-level)
│   ├── Dashboard/
│   ├── UserProfile/
│   └── Settings/
├── features/           # Feature-based modules
│   ├── auth/
│   ├── users/
│   └── notifications/
├── hooks/              # Custom React hooks
├── services/           # API calls and external services
├── store/              # State management (Redux/Zustand/Context)
├── types/              # TypeScript type definitions
├── utils/              # Utility functions and helpers
├── theme/              # MUI theme configuration
├── routes/             # Route definitions
└── config/             # App configuration
```

## Core Principles

### 1. Page-Based Organization

Each page should be a standalone module in the `pages/` directory. Pages are top-level components that correspond to routes.

```typescript
// pages/Dashboard/Dashboard.tsx
import { FC } from 'react';
import { Container, Grid } from '@mui/material';
import { DashboardStats } from './components/DashboardStats';
import { RecentActivity } from './components/RecentActivity';

export const Dashboard: FC = () => {
  return (
    <Container maxWidth="lg">
      <Grid container spacing={3}>
        <Grid item xs={12} md={8}>
          <DashboardStats />
        </Grid>
        <Grid item xs={12} md={4}>
          <RecentActivity />
        </Grid>
      </Grid>
    </Container>
  );
};
```

### 2. Component Categories

**Common Components** (`components/common/`): Highly reusable, generic UI elements used across the application.

**Layout Components** (`components/layout/`): Structural components that define page layouts, headers, sidebars, and navigation.

**Page-Specific Components**: Store these within the page directory they belong to, not in the global components folder.

```
pages/
└── Dashboard/
    ├── Dashboard.tsx
    ├── components/
    │   ├── DashboardStats.tsx
    │   └── RecentActivity.tsx
    └── index.ts
```

### 3. Feature Modules

For complex features with multiple related components, services, and state, use the `features/` directory.

```
features/
└── auth/
    ├── components/
    │   ├── LoginForm.tsx
    │   └── RegisterForm.tsx
    ├── hooks/
    │   └── useAuth.ts
    ├── services/
    │   └── authService.ts
    ├── store/
    │   └── authSlice.ts
    └── types/
        └── auth.types.ts
```

## TypeScript Best Practices

### Type Definitions

Create interfaces and types in dedicated files. Use `.types.ts` suffix for type-only files.

```typescript
// types/user.types.ts
export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
}

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest',
}

export type UserFormData = Omit<User, 'id'>;
```

### Component Typing

Always type component props explicitly.

```typescript
// components/common/UserCard/UserCard.tsx
import { FC } from 'react';
import { Card, CardContent, Typography } from '@mui/material';
import { User } from '@/types/user.types';

interface UserCardProps {
  user: User;
  onEdit?: (userId: string) => void;
  className?: string;
}

export const UserCard: FC<UserCardProps> = ({ user, onEdit, className }) => {
  return (
    <Card className={className}>
      <CardContent>
        <Typography variant="h6">
          {user.firstName} {user.lastName}
        </Typography>
        <Typography color="text.secondary">{user.email}</Typography>
      </CardContent>
    </Card>
  );
};
```

## Material-UI Integration

### Theme Configuration

Centralize theme customization in a dedicated theme file.

```typescript
// theme/theme.ts
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
      light: '#42a5f5',
      dark: '#1565c0',
    },
    secondary: {
      main: '#9c27b0',
    },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 500,
    },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          borderRadius: 8,
        },
      },
    },
  },
});
```

### Consistent Spacing

Use MUI's spacing system consistently throughout the application.

```typescript
<Box sx={{ p: 3, mb: 2 }}>  // padding: 24px, margin-bottom: 16px
  <Stack spacing={2}>       // gap: 16px between children
    <TextField />
    <Button />
  </Stack>
</Box>
```

## Routing

Use React Router with lazy loading for code splitting.

```typescript
// routes/routes.tsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter } from 'react-router-dom';
import { MainLayout } from '@/components/layout/MainLayout';
import { LoadingSpinner } from '@/components/common/LoadingSpinner';

const Dashboard = lazy(() => import('@/pages/Dashboard'));
const UserProfile = lazy(() => import('@/pages/UserProfile'));

const LazyLoad = ({ children }: { children: React.ReactNode }) => (
  <Suspense fallback={<LoadingSpinner />}>{children}</Suspense>
);

export const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <LazyLoad><Dashboard /></LazyLoad>,
      },
      {
        path: 'profile/:userId',
        element: <LazyLoad><UserProfile /></LazyLoad>,
      },
    ],
  },
]);
```

## State Management

### Local State

Use React hooks for component-level state.

```typescript
const [isOpen, setIsOpen] = useState(false);
const [users, setUsers] = useState<User[]>([]);
```

### Global State

For application-wide state, choose based on complexity: Context API for simple cases, Zustand or Redux Toolkit for complex applications.

```typescript
// store/userStore.ts (Zustand example)
import { create } from 'zustand';
import { User } from '@/types/user.types';

interface UserState {
  currentUser: User | null;
  setCurrentUser: (user: User | null) => void;
}

export const useUserStore = create<UserState>((set) => ({
  currentUser: null,
  setCurrentUser: (user) => set({ currentUser: user }),
}));
```

## Custom Hooks

Extract reusable logic into custom hooks.

```typescript
// hooks/useDebounce.ts
import { useEffect, useState } from 'react';

export const useDebounce = <T,>(value: T, delay: number = 500): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
};
```

## API Services

Centralize API calls in service files.

```typescript
// services/userService.ts
import axios from 'axios';
import { User, UserFormData } from '@/types/user.types';

const API_BASE_URL = process.env.REACT_APP_API_URL;

export const userService = {
  getUsers: async (): Promise<User[]> => {
    const response = await axios.get(`${API_BASE_URL}/users`);
    return response.data;
  },

  getUserById: async (id: string): Promise<User> => {
    const response = await axios.get(`${API_BASE_URL}/users/${id}`);
    return response.data;
  },

  createUser: async (data: UserFormData): Promise<User> => {
    const response = await axios.post(`${API_BASE_URL}/users`, data);
    return response.data;
  },
};
```

## Form Handling

Use libraries like React Hook Form with MUI for form management.

```typescript
// pages/UserCreate/components/UserForm.tsx
import { FC } from 'react';
import { useForm, Controller } from 'react-hook-form';
import { TextField, Button, Stack } from '@mui/material';
import { UserFormData } from '@/types/user.types';

interface UserFormProps {
  onSubmit: (data: UserFormData) => void;
}

export const UserForm: FC<UserFormProps> = ({ onSubmit }) => {
  const { control, handleSubmit } = useForm<UserFormData>();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Stack spacing={2}>
        <Controller
          name="firstName"
          control={control}
          rules={{ required: 'First name is required' }}
          render={({ field, fieldState }) => (
            <TextField
              {...field}
              label="First Name"
              error={!!fieldState.error}
              helperText={fieldState.error?.message}
            />
          )}
        />
        <Button type="submit" variant="contained">
          Submit
        </Button>
      </Stack>
    </form>
  );
};
```

## Error Handling

Implement error boundaries and consistent error handling patterns.

```typescript
// components/common/ErrorBoundary/ErrorBoundary.tsx
import { Component, ErrorInfo, ReactNode } from 'react';
import { Alert, Button, Box } from '@mui/material';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <Box sx={{ p: 3 }}>
          <Alert severity="error">
            Something went wrong. Please try again.
          </Alert>
          <Button onClick={() => this.setState({ hasError: false })}>
            Reset
          </Button>
        </Box>
      );
    }

    return this.props.children;
  }
}
```

## Testing

Organize tests alongside the components they test.

```
components/
└── common/
    └── UserCard/
        ├── UserCard.tsx
        ├── UserCard.test.tsx
        └── index.ts
```

## File Naming Conventions

- **Components**: PascalCase (UserCard.tsx)
- **Utilities**: camelCase (formatDate.ts)
- **Types**: camelCase with .types.ts suffix (user.types.ts)
- **Hooks**: camelCase with use prefix (useAuth.ts)
- **Constants**: UPPER_SNAKE_CASE in constants.ts files

## Index Files

Use index.ts files for cleaner imports.

```typescript
// components/common/index.ts
export { UserCard } from './UserCard/UserCard';
export { LoadingSpinner } from './LoadingSpinner/LoadingSpinner';
export { ErrorMessage } from './ErrorMessage/ErrorMessage';

// Usage:
import { UserCard, LoadingSpinner } from '@/components/common';
```

## Environment Configuration

```typescript
// config/env.ts
export const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3000',
  environment: process.env.NODE_ENV,
  features: {
    enableAnalytics: process.env.REACT_APP_ENABLE_ANALYTICS === 'true',
  },
} as const;
```

This structure provides a solid foundation for scalable, maintainable React applications while leveraging TypeScript's type safety and Material-UI's comprehensive component library.
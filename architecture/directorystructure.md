# Directory Structure for Full-Stack Applications

## Table of Contents
- [Introduction](#introduction)
- [Repository Organization](#repository-organization)
- [Backend Structure (Java/Spring)](#backend-structure-javaspring)
- [Frontend Structure (React/TypeScript)](#frontend-structure-reacttypescript)
- [Testing Organization](#testing-organization)
- [Shared Resources](#shared-resources)
- [Scaling Strategies](#scaling-strategies)
- [Module Organization Patterns](#module-organization-patterns)
- [Build and Deployment Structure](#build-and-deployment-structure)
- [Common Anti-Patterns](#common-anti-patterns)

---

## Introduction

Directory structure significantly impacts code maintainability, team collaboration, and application scalability. A well-organized structure enables engineers to locate code quickly, understand system architecture, and modify features without affecting unrelated code.

This guide addresses structure for applications as they grow to include:
- Multiple features and bounded contexts
- Complex forms with nested subforms
- Numerous pages and routes
- Comprehensive test coverage
- Shared components and utilities
- Multiple deployment environments

### Technology Stack

This guide is tailored for the following tech stack:
- **Backend:** Java 21 with Spring Boot 3.4.x, Gradle build system, PostgreSQL database
- **Frontend:** React 18+ with TypeScript, MVVM architecture pattern, Yarn package manager, Vite build tool
- **Testing:** JUnit 5/Mockito (backend), Vitest/React Testing Library (frontend), Playwright (E2E)

### Key Principles

**Separation by Feature:** Related code is grouped by feature or domain rather than technical layer.

**MVVM Architecture:** The frontend follows the Model-View-ViewModel pattern with custom hooks serving as ViewModels.

**Colocation:** Related files are placed close together. Tests, types, and styles live near the code they support.

**Discoverability:** The structure makes it obvious where new code belongs and where existing code lives.

**Scalability:** The organization accommodates growth without requiring major restructuring.

---

## Repository Organization

This guide assumes a monorepo containing both the frontend and backend in a single repository.

```
project-root/
├── backend/                 # Java/Spring application
├── frontend/                # React/TypeScript application
├── docs/                    # Project documentation
├── scripts/                 # Build and deployment scripts
├── .github/                 # GitHub workflows and templates
│   └── workflows/
├── docker-compose.yml       # Local development environment
├── README.md
└── .gitignore
```

**Advantages of a monorepo:**
- Simplified dependency management across frontend and backend
- Atomic commits when changes span both layers
- Easier code sharing and coordinated refactoring
- Single CI/CD pipeline with unified versioning
- Reduced coordination overhead for breaking API changes

**Considerations to manage:**
- Repository size grows with both codebases
- Build configuration requires attention to both stacks
- Path-based CI triggers should be used to avoid unnecessary builds

---

## Backend Structure (Java/Spring)

### Standard Gradle Project Layout

```
backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── todos/
│   │   │               ├── TodoApplication.java
│   │   │               ├── config/              # Application configuration
│   │   │               │   ├── SecurityConfig.java
│   │   │               │   ├── DatabaseConfig.java
│   │   │               │   ├── CorsConfig.java
│   │   │               │   └── OpenApiConfig.java
│   │   │               ├── common/              # Shared utilities
│   │   │               │   ├── exception/
│   │   │               │   │   ├── GlobalExceptionHandler.java
│   │   │               │   │   ├── ResourceNotFoundException.java
│   │   │               │   │   ├── BusinessRuleException.java
│   │   │               │   │   └── ValidationException.java
│   │   │               │   ├── util/
│   │   │               │   │   ├── DateUtils.java
│   │   │               │   │   ├── StringUtils.java
│   │   │               │   │   └── ValidationUtils.java
│   │   │               │   └── constants/
│   │   │               │       ├── ErrorMessages.java
│   │   │               │       └── AppConstants.java
│   │   │               ├── todo/                # Feature-based modules
│   │   │               │   ├── controller/
│   │   │               │   │   └── TodoController.java
│   │   │               │   ├── dto/
│   │   │               │   │   ├── CreateTodoRequest.java
│   │   │               │   │   ├── UpdateTodoRequest.java
│   │   │               │   │   ├── TodoResponse.java
│   │   │               │   │   └── TodoSummary.java
│   │   │               │   ├── entity/
│   │   │               │   │   ├── TodoEntity.java
│   │   │               │   │   └── TodoPriority.java
│   │   │               │   ├── repository/
│   │   │               │   │   └── TodoRepository.java
│   │   │               │   ├── service/
│   │   │               │   │   ├── TodoService.java
│   │   │               │   │   └── TodoValidationService.java
│   │   │               │   └── mapper/
│   │   │               │       └── TodoMapper.java
│   │   │               ├── category/
│   │   │               │   ├── controller/
│   │   │               │   ├── dto/
│   │   │               │   ├── entity/
│   │   │               │   ├── repository/
│   │   │               │   ├── service/
│   │   │               │   └── mapper/
│   │   │               └── security/            # Authentication/Authorization
│   │   │                   ├── JwtTokenProvider.java
│   │   │                   ├── UserDetailsServiceImpl.java
│   │   │                   └── AuthenticationController.java
│   │   └── resources/
│   │       ├── application.yml              # Main configuration
│   │       ├── application-dev.yml          # Development profile
│   │       ├── application-test.yml         # Test profile
│   │       ├── application-prod.yml         # Production profile
│   │       ├── db/
│   │       │   └── migration/               # Flyway migrations
│   │       │       ├── V1__initial_schema.sql
│   │       │       ├── V2__add_users_table.sql
│   │       │       └── V3__add_todo_tables.sql
│   │       ├── static/                      # Static resources (if any)
│   │       └── templates/                   # Email templates, etc.
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── example/
│       │           └── todos/
│       │               ├── todo/
│       │               │   ├── controller/
│       │               │   │   └── TodoControllerTest.java
│       │               │   ├── service/
│       │               │   │   └── TodoServiceTest.java
│       │               │   ├── repository/
│       │               │   │   └── TodoRepositoryTest.java
│       │               │   └── integration/
│       │               │       └── TodoIntegrationTest.java
│       │               └── common/
│       │                   └── BaseIntegrationTest.java
│       └── resources/
│           ├── application-test.yml
│           └── test-data/
│               └── todo-test-data.sql
├── build/                                   # Build output (gitignored)
│   ├── classes/
│   ├── libs/
│   └── reports/
├── gradle/                                  # Gradle wrapper
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── build.gradle.kts                         # Gradle build configuration (Kotlin DSL)
├── settings.gradle.kts                      # Gradle settings
├── gradlew                                  # Gradle wrapper script (Unix)
├── gradlew.bat                              # Gradle wrapper script (Windows)
├── .gitignore
└── README.md
```

### Feature Module Deep Dive

For complex features with multiple subdomains, the organization is as follows:

```
backend/src/main/java/com/example/todos/
└── todo/                                    # Feature module
    ├── controller/
    │   ├── TodoController.java              # Main todo operations
    │   ├── CategoryController.java          # Category management
    │   └── CommentController.java           # Comments on todos
    ├── dto/
    │   ├── todo/
    │   │   ├── CreateTodoRequest.java
    │   │   ├── UpdateTodoRequest.java
    │   │   ├── TodoResponse.java
    │   │   └── TodoSummary.java
    │   ├── category/
    │   │   ├── CategoryRequest.java
    │   │   └── CategoryResponse.java
    │   └── comment/
    │       ├── CommentRequest.java
    │       └── CommentResponse.java
    ├── entity/
    │   ├── TodoEntity.java
    │   ├── CategoryEntity.java
    │   ├── CommentEntity.java
    │   └── TodoPriority.java                # Enums
    ├── repository/
    │   ├── TodoRepository.java
    │   ├── CategoryRepository.java
    │   ├── CommentRepository.java
    │   └── specification/                   # Custom queries
    │       └── TodoSpecification.java
    ├── service/
    │   ├── TodoService.java
    │   ├── CategoryService.java
    │   ├── CommentService.java
    │   └── impl/                            # Service implementations if using interfaces
    │       ├── TodoServiceImpl.java
    │       ├── CategoryServiceImpl.java
    │       └── CommentServiceImpl.java
    └── mapper/
        ├── TodoMapper.java
        ├── CategoryMapper.java
        └── CommentMapper.java
```

### Configuration Organization

```
backend/src/main/java/com/example/todos/config/
├── SecurityConfig.java                      # Spring Security configuration
├── WebConfig.java                           # Web MVC configuration
├── DatabaseConfig.java                      # DataSource and JPA configuration
├── CacheConfig.java                         # Redis/Caffeine cache configuration
├── AsyncConfig.java                         # Async execution configuration
├── OpenApiConfig.java                       # OpenAPI/Swagger documentation
├── CorsConfig.java                          # CORS policy
└── properties/                              # Type-safe configuration properties
    ├── AppProperties.java
    ├── JwtProperties.java
    └── StorageProperties.java
```

### Gradle Configuration Example

**build.gradle.kts (Kotlin DSL - recommended for type safety):**

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.4.1"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.example"
version = "1.0.0"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

configurations {
    compileOnly {
        extendsFrom(configurations.annotationProcessor.get())
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot starters
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    
    // Database
    runtimeOnly("org.postgresql:postgresql")
    implementation("org.flywaydb:flyway-core")
    implementation("org.flywaydb:flyway-database-postgresql")
    
    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    // MapStruct
    implementation("org.mapstruct:mapstruct:1.6.3")
    annotationProcessor("org.mapstruct:mapstruct-processor:1.6.3")
    annotationProcessor("org.projectlombok:lombok-mapstruct-binding:0.2.0")
    
    // OpenAPI documentation
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.7.0")
    
    // Testing
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
    testImplementation("org.testcontainers:postgresql")
    testImplementation("org.testcontainers:junit-jupiter")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

**settings.gradle.kts:**

```kotlin
rootProject.name = "todos"
```

---

## Frontend Structure (React/TypeScript)

### Feature-Based Organization with MVVM

This structure implements MVVM architecture where custom hooks serve as ViewModels, managing presentation logic and state.

```
frontend/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── assets/                              # Static assets
│       └── images/
├── src/
│   ├── App.tsx                              # Root component
│   ├── main.tsx                             # Entry point
│   ├── app/                                 # App-level configuration
│   │   ├── store/                           # Redux/state management (Model layer)
│   │   │   ├── index.ts
│   │   │   ├── rootReducer.ts
│   │   │   └── hooks.ts
│   │   ├── routes/                          # Routing configuration
│   │   │   ├── index.tsx
│   │   │   ├── AppRoutes.tsx
│   │   │   ├── PrivateRoute.tsx
│   │   │   └── RouteConfig.ts
│   │   └── providers/                       # Context providers
│   │       ├── AuthProvider.tsx
│   │       └── ThemeProvider.tsx
│   ├── features/                            # Feature modules
│   │   ├── auth/                            # Authentication feature
│   │   │   ├── components/                  # View layer
│   │   │   │   ├── LoginForm/
│   │   │   │   │   ├── LoginForm.tsx
│   │   │   │   │   ├── LoginForm.test.tsx
│   │   │   │   │   ├── LoginForm.module.css
│   │   │   │   │   └── index.ts
│   │   │   │   ├── RegisterForm/
│   │   │   │   └── PasswordResetForm/
│   │   │   ├── pages/                       # View layer (page components)
│   │   │   │   ├── LoginPage.tsx
│   │   │   │   ├── RegisterPage.tsx
│   │   │   │   └── PasswordResetPage.tsx
│   │   │   ├── viewmodels/                  # ViewModel layer (presentation logic)
│   │   │   │   ├── useLoginViewModel.ts     # Login ViewModel
│   │   │   │   ├── useRegisterViewModel.ts  # Register ViewModel
│   │   │   │   └── useAuthViewModel.ts      # Shared auth ViewModel
│   │   │   ├── hooks/                       # Additional custom hooks
│   │   │   │   └── usePasswordStrength.ts
│   │   │   ├── models/                      # Model layer (business logic)
│   │   │   │   ├── services/
│   │   │   │   │   └── authService.ts
│   │   │   │   ├── store/
│   │   │   │   │   ├── authSlice.ts
│   │   │   │   │   └── authSelectors.ts
│   │   │   │   └── validation/
│   │   │   │       └── authValidation.ts
│   │   │   ├── types/
│   │   │   │   ├── auth.types.ts
│   │   │   │   └── user.types.ts
│   │   │   ├── utils/
│   │   │   │   └── tokenUtils.ts
│   │   │   └── index.ts
│   │   ├── todos/                           # Todo management feature
│   │   │   ├── components/                  # View layer
│   │   │   │   ├── TodoList/
│   │   │   │   │   ├── TodoList.tsx
│   │   │   │   │   ├── TodoList.test.tsx
│   │   │   │   │   ├── TodoList.module.css
│   │   │   │   │   └── index.ts
│   │   │   │   ├── TodoItem/
│   │   │   │   ├── TodoForm/
│   │   │   │   │   ├── TodoForm.tsx
│   │   │   │   │   ├── TodoForm.test.tsx
│   │   │   │   │   ├── subforms/           # Nested subforms
│   │   │   │   │   │   ├── TodoDetailsSubform.tsx
│   │   │   │   │   │   ├── TodoCategorySubform.tsx
│   │   │   │   │   │   └── TodoAttachmentsSubform.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   ├── TodoFilters/
│   │   │   │   └── TodoStats/
│   │   │   ├── pages/                       # View layer (page components)
│   │   │   │   ├── TodoListPage.tsx
│   │   │   │   ├── TodoDetailPage.tsx
│   │   │   │   └── TodoEditPage.tsx
│   │   │   ├── viewmodels/                  # ViewModel layer
│   │   │   │   ├── useTodoListViewModel.ts  # List ViewModel
│   │   │   │   ├── useTodoFormViewModel.ts  # Form ViewModel
│   │   │   │   ├── useTodoDetailViewModel.ts # Detail ViewModel
│   │   │   │   └── useTodoFiltersViewModel.ts # Filters ViewModel
│   │   │   ├── hooks/                       # Additional custom hooks
│   │   │   │   ├── useTodoPagination.ts
│   │   │   │   └── useTodoSort.ts
│   │   │   ├── models/                      # Model layer
│   │   │   │   ├── services/
│   │   │   │   │   ├── todoService.ts
│   │   │   │   │   └── categoryService.ts
│   │   │   │   ├── store/
│   │   │   │   │   ├── todoSlice.ts
│   │   │   │   │   ├── todoSelectors.ts
│   │   │   │   │   └── todoThunks.ts
│   │   │   │   └── validation/
│   │   │   │       └── todoValidation.ts
│   │   │   ├── types/
│   │   │   │   ├── todo.types.ts
│   │   │   │   ├── category.types.ts
│   │   │   │   └── api.types.ts
│   │   │   ├── utils/
│   │   │   │   └── todoHelpers.ts
│   │   │   └── index.ts
│   │   ├── dashboard/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── viewmodels/
│   │   │   │   └── useDashboardViewModel.ts
│   │   │   ├── models/
│   │   │   └── widgets/                     # Dashboard-specific widgets
│   │   │       ├── RecentTodosWidget/
│   │   │       ├── TodoStatsWidget/
│   │   │       └── ActivityFeedWidget/
│   │   └── profile/
│   │       ├── components/
│   │       ├── pages/
│   │       ├── viewmodels/
│   │       │   └── useProfileViewModel.ts
│   │       ├── models/
│   │       │   └── services/
│   │       │       └── profileService.ts
│   │       └── hooks/
│   ├── shared/                              # Shared across features
│   │   ├── components/                      # Reusable UI components
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Button.test.tsx
│   │   │   │   ├── Button.module.css
│   │   │   │   └── index.ts
│   │   │   ├── Input/
│   │   │   ├── Select/
│   │   │   ├── Modal/
│   │   │   ├── Table/
│   │   │   ├── Pagination/
│   │   │   ├── DatePicker/
│   │   │   ├── FormField/
│   │   │   ├── LoadingSpinner/
│   │   │   ├── ErrorBoundary/
│   │   │   └── Layout/
│   │   │       ├── Header/
│   │   │       ├── Sidebar/
│   │   │       ├── Footer/
│   │   │       └── MainLayout.tsx
│   │   ├── hooks/                           # Shared custom hooks
│   │   │   ├── useDebounce.ts
│   │   │   ├── useLocalStorage.ts
│   │   │   ├── useMediaQuery.ts
│   │   │   ├── usePagination.ts
│   │   │   └── useForm.ts
│   │   ├── utils/                           # Utility functions
│   │   │   ├── api/
│   │   │   │   ├── apiClient.ts
│   │   │   │   ├── interceptors.ts
│   │   │   │   └── errorHandling.ts
│   │   │   ├── validation/
│   │   │   │   ├── validators.ts
│   │   │   │   └── schemas.ts
│   │   │   ├── formatting/
│   │   │   │   ├── dateFormatters.ts
│   │   │   │   ├── numberFormatters.ts
│   │   │   │   └── stringFormatters.ts
│   │   │   └── helpers/
│   │   │       ├── arrayHelpers.ts
│   │   │       └── objectHelpers.ts
│   │   ├── types/                           # Shared type definitions
│   │   │   ├── api.types.ts
│   │   │   ├── common.types.ts
│   │   │   └── dto.types.ts
│   │   ├── constants/
│   │   │   ├── routes.ts
│   │   │   ├── apiEndpoints.ts
│   │   │   └── appConstants.ts
│   │   └── styles/                          # Global styles
│   │       ├── variables.css
│   │       ├── mixins.css
│   │       ├── reset.css
│   │       └── global.css
│   ├── assets/                              # Images, fonts, etc.
│   │   ├── images/
│   │   ├── fonts/
│   │   └── icons/
│   └── types/                               # Global TypeScript definitions
│       ├── env.d.ts
│       └── global.d.ts
├── tests/                                   # Additional test utilities
│   ├── setup.ts
│   ├── mocks/
│   │   ├── handlers.ts                      # MSW handlers
│   │   └── server.ts
│   └── utils/
│       ├── testUtils.tsx
│       └── renderWithProviders.tsx
├── .env                                     # Environment variables
├── .env.development
├── .env.production
├── .yarn/                                   # Yarn cache (gitignored)
├── .yarnrc.yml                              # Yarn configuration
├── package.json
├── yarn.lock                                # Yarn lock file
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts                         # Vitest configuration
├── playwright.config.ts                     # E2E test configuration
├── eslint.config.js                         # ESLint flat config
├── .prettierrc
├── .gitignore
└── README.md
```

### Complex Form Organization

For features with complex nested forms:

```
src/features/orders/
└── components/
    └── OrderForm/
        ├── OrderForm.tsx                    # Main form component (View)
        ├── OrderForm.test.tsx
        ├── OrderForm.types.ts               # Form-specific types
        ├── OrderForm.validation.ts          # Validation schemas (Model)
        ├── OrderForm.module.css
        ├── subforms/                        # Nested subforms (View)
        │   ├── CustomerInfoSubform/
        │   │   ├── CustomerInfoSubform.tsx
        │   │   ├── CustomerInfoSubform.test.tsx
        │   │   └── index.ts
        │   ├── ShippingAddressSubform/
        │   │   ├── ShippingAddressSubform.tsx
        │   │   ├── ShippingAddressSubform.test.tsx
        │   │   └── index.ts
        │   ├── OrderItemsSubform/
        │   │   ├── OrderItemsSubform.tsx
        │   │   ├── OrderItemsSubform.test.tsx
        │   │   ├── components/
        │   │   │   └── OrderItemRow.tsx     # Sub-subform component
        │   │   └── index.ts
        │   └── PaymentInfoSubform/
        │       ├── PaymentInfoSubform.tsx
        │       ├── PaymentInfoSubform.test.tsx
        │       └── index.ts
        └── index.ts

src/features/orders/
└── viewmodels/                              # ViewModel layer for form
    ├── useOrderFormViewModel.ts             # Main form ViewModel
    ├── useOrderFormViewModel.test.ts
    └── subforms/                            # Subform ViewModels
        ├── useCustomerInfoViewModel.ts
        ├── useShippingAddressViewModel.ts
        ├── useOrderItemsViewModel.ts
        └── usePaymentInfoViewModel.ts

src/features/orders/
└── models/                                  # Model layer for form
    ├── services/
    │   └── orderService.ts                  # Business logic
    ├── validation/
    │   ├── orderValidation.ts               # Validation rules
    │   ├── customerValidation.ts
    │   └── paymentValidation.ts
    └── store/
        └── orderSlice.ts                    # State management
```

### MVVM Architecture in React

The frontend follows MVVM principles with React-specific implementation:

**Model Layer:**
- `models/services/` - API calls and business logic
- `models/store/` - State management (Redux slices)
- `models/validation/` - Business rules and validation

**ViewModel Layer:**
- `viewmodels/` - Custom hooks that manage presentation logic
- Expose data in View-friendly format
- Handle user interactions
- Orchestrate Model operations
- Manage component state

**View Layer:**
- `components/` - React components (UI only)
- `pages/` - Page-level components
- Bind to ViewModel via hooks
- Minimal logic, primarily rendering

**Example ViewModel Structure:**

```typescript
// src/features/todos/viewmodels/useTodoListViewModel.ts
import { useState, useEffect, useMemo, useCallback } from 'react';
import { useAppDispatch, useAppSelector } from '@app/store/hooks';
import { fetchTodos, selectFilteredTodos, selectTodosLoading } from '../models/store/todoSlice';
import { todoService } from '../models/services/todoService';
import type { Todo, TodoFilter } from '../types/todo.types';

export function useTodoListViewModel() {
  // State from Model layer
  const dispatch = useAppDispatch();
  const todos = useAppSelector(selectFilteredTodos);
  const loading = useAppSelector(selectTodosLoading);
  
  // ViewModel-specific state
  const [filter, setFilter] = useState<TodoFilter>('all');
  const [searchTerm, setSearchTerm] = useState('');
  
  // Computed properties for View
  const filteredTodos = useMemo(() => {
    return todos.filter(todo => 
      todo.text.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [todos, searchTerm]);
  
  const activeCount = useMemo(() => {
    return todos.filter(t => !t.completed).length;
  }, [todos]);
  
  const completedCount = useMemo(() => {
    return todos.filter(t => t.completed).length;
  }, [todos]);
  
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
    dispatch(fetchTodos());
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
    
    // Commands
    loadTodos,
    handleFilterChange,
    handleSearchChange,
    deleteTodo,
  };
}
```

**Example View Using ViewModel:**

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
    <div>
      <input
        type="text"
        value={viewModel.searchTerm}
        onChange={(e) => viewModel.handleSearchChange(e.target.value)}
        placeholder="Search todos..."
      />
      
      <div>
        <button onClick={() => viewModel.handleFilterChange('all')}>
          All ({viewModel.todos.length})
        </button>
        <button onClick={() => viewModel.handleFilterChange('active')}>
          Active ({viewModel.activeCount})
        </button>
        <button onClick={() => viewModel.handleFilterChange('completed')}>
          Completed ({viewModel.completedCount})
        </button>
      </div>
      
      <ul>
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

### Page Organization for Large Applications

```
src/features/todos/pages/
├── TodoListPage/
│   ├── TodoListPage.tsx
│   ├── TodoListPage.test.tsx
│   ├── components/                          # Page-specific components
│   │   ├── TodoListHeader.tsx
│   │   ├── TodoListToolbar.tsx
│   │   └── EmptyTodosPlaceholder.tsx
│   ├── hooks/
│   │   ├── useTodoListPage.ts               # Page-specific logic
│   │   └── useTodoListFilters.ts
│   └── index.ts
├── TodoDetailPage/
│   ├── TodoDetailPage.tsx
│   ├── TodoDetailPage.test.tsx
│   ├── components/
│   │   ├── TodoHeader.tsx
│   │   ├── TodoMetadata.tsx
│   │   ├── TodoComments.tsx
│   │   └── TodoActivityLog.tsx
│   ├── hooks/
│   │   └── useTodoDetailPage.ts
│   └── index.ts
└── TodoEditPage/
    ├── TodoEditPage.tsx
    ├── TodoEditPage.test.tsx
    └── index.ts
```

### Frontend Package Configuration

**package.json (with Yarn):**

```json
{
  "name": "todos-frontend",
  "version": "1.0.0",
  "type": "module",
  "packageManager": "yarn@4.5.3",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:watch": "vitest watch",
    "test:coverage": "vitest --coverage",
    "test:viewmodels": "vitest --testPathPattern=viewmodels",
    "test:components": "vitest --testPathPattern=components",
    "test:e2e": "playwright test",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^7.1.1",
    "@reduxjs/toolkit": "^2.5.0",
    "react-redux": "^9.2.0",
    "axios": "^1.7.9",
    "zod": "^3.24.1",
    "date-fns": "^4.1.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.17",
    "@types/react-dom": "^18.3.5",
    "@testing-library/react": "^16.1.0",
    "@testing-library/jest-dom": "^6.6.3",
    "@testing-library/user-event": "^14.5.2",
    "typescript": "~5.7.2",
    "vite": "^6.0.7",
    "@vitejs/plugin-react": "^4.3.4",
    "vitest": "^2.1.8",
    "@vitest/coverage-v8": "^2.1.8",
    "jsdom": "^25.0.1",
    "eslint": "^9.17.0",
    "prettier": "^3.4.2",
    "msw": "^2.7.0",
    "@playwright/test": "^1.49.1"
  }
}
```

---

## Testing Organization

### Backend Testing Structure

```
backend/src/test/java/com/example/todos/
├── todo/
│   ├── controller/
│   │   └── TodoControllerTest.java          # Controller layer tests
│   ├── service/
│   │   └── TodoServiceTest.java             # Service layer unit tests
│   ├── repository/
│   │   └── TodoRepositoryTest.java          # Repository tests
│   ├── mapper/
│   │   └── TodoMapperTest.java              # Mapper tests
│   └── integration/
│       ├── TodoIntegrationTest.java         # End-to-end feature tests
│       └── TodoApiTest.java                 # API contract tests
├── common/
│   ├── BaseIntegrationTest.java             # Base class for integration tests
│   ├── TestDataBuilder.java                 # Test data builders
│   └── TestUtils.java
└── testcontainers/                          # Testcontainers configuration
    ├── PostgresTestContainer.java
    └── RedisTestContainer.java
```

**Test Resource Organization:**

```
backend/src/test/resources/
├── application-test.yml                     # Test configuration
├── test-data/
│   ├── users.sql                            # Test data SQL
│   ├── todos.sql
│   └── categories.sql
├── fixtures/                                # JSON test fixtures
│   ├── valid-todo-request.json
│   └── invalid-todo-request.json
└── contracts/                               # API contract definitions
    └── todo-api-contract.json
```

### Frontend Testing Structure

```
src/features/todos/
├── components/
│   ├── TodoList/
│   │   ├── TodoList.tsx
│   │   ├── TodoList.test.tsx                # Component unit tests (View)
│   │   └── TodoList.integration.test.tsx    # Component integration tests
│   └── TodoForm/
│       ├── TodoForm.tsx
│       ├── TodoForm.test.tsx
│       └── subforms/
│           └── TodoDetailsSubform/
│               ├── TodoDetailsSubform.tsx
│               └── TodoDetailsSubform.test.tsx
├── viewmodels/
│   ├── useTodoListViewModel.ts
│   ├── useTodoListViewModel.test.ts         # ViewModel tests
│   ├── useTodoFormViewModel.ts
│   └── useTodoFormViewModel.test.ts
├── hooks/
│   ├── useTodoPagination.ts
│   └── useTodoPagination.test.ts            # Hook tests
├── models/
│   ├── services/
│   │   ├── todoService.ts
│   │   └── todoService.test.ts              # Service tests (Model)
│   ├── store/
│   │   ├── todoSlice.ts
│   │   └── todoSlice.test.ts                # Redux slice tests (Model)
│   └── validation/
│       ├── todoValidation.ts
│       └── todoValidation.test.ts           # Validation tests (Model)
└── __tests__/                               # Feature-level tests
    ├── todos.integration.test.tsx           # Feature integration tests
    └── todos.e2e.test.tsx                   # E2E tests (or in separate e2e/ directory)
```

**Global Test Setup:**

```
tests/
├── setup.ts                                 # Global test setup
├── mocks/
│   ├── handlers/                            # MSW request handlers
│   │   ├── authHandlers.ts
│   │   ├── todoHandlers.ts
│   │   └── index.ts
│   └── server.ts                            # MSW server setup
├── utils/
│   ├── testUtils.tsx                        # Custom render functions
│   ├── renderWithProviders.tsx              # Render with Redux/Router
│   ├── mockViewModels.ts                    # Mock ViewModel factories
│   └── mockData.ts                          # Mock data generators
└── fixtures/
    ├── todos.ts                             # Test data fixtures
    └── users.ts
```

**ViewModel Testing Example:**

```typescript
// src/features/todos/viewmodels/useTodoListViewModel.test.ts
import { renderHook, act, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { useTodoListViewModel } from './useTodoListViewModel';
import { todoSlice } from '../models/store/todoSlice';
import * as todoService from '../models/services/todoService';

vi.mock('../models/services/todoService');

describe('useTodoListViewModel', () => {
  let store;
  
  beforeEach(() => {
    store = configureStore({
      reducer: { todos: todoSlice.reducer }
    });
  });
  
  it('loads todos on mount', async () => {
    const mockTodos = [
      { id: '1', text: 'Todo 1', completed: false },
      { id: '2', text: 'Todo 2', completed: true }
    ];
    
    vi.mocked(todoService.getTodos).mockResolvedValue(mockTodos);
    
    const wrapper = ({ children }) => (
      <Provider store={store}>{children}</Provider>
    );
    
    const { result } = renderHook(() => useTodoListViewModel(), { wrapper });
    
    await waitFor(() => {
      expect(result.current.todos).toHaveLength(2);
    });
  });
  
  it('filters todos by search term', async () => {
    const wrapper = ({ children }) => (
      <Provider store={store}>{children}</Provider>
    );
    
    const { result } = renderHook(() => useTodoListViewModel(), { wrapper });
    
    act(() => {
      result.current.handleSearchChange('Todo 1');
    });
    
    expect(result.current.searchTerm).toBe('Todo 1');
  });
  
  it('calculates active and completed counts correctly', () => {
    // Test computed properties
  });
});
```

**E2E Testing Structure (Playwright):**

```
frontend/
└── e2e/
    ├── fixtures/
    │   ├── todos.json
    │   └── users.json
    ├── support/
    │   ├── commands.ts                      # Custom commands
    │   └── index.ts
    └── tests/
        ├── auth/
        │   ├── login.spec.ts
        │   └── registration.spec.ts
        ├── todos/
        │   ├── todo-creation.spec.ts
        │   ├── todo-editing.spec.ts
        │   └── todo-filtering.spec.ts
        └── workflows/
            └── complete-todo-workflow.spec.ts
```

---

## Shared Resources

### API Client Organization

```
src/shared/utils/api/
├── apiClient.ts                             # Axios/Fetch wrapper
├── interceptors/
│   ├── authInterceptor.ts                   # Add auth tokens
│   ├── errorInterceptor.ts                  # Global error handling
│   └── loggingInterceptor.ts                # Request/response logging
├── endpoints/
│   ├── authEndpoints.ts                     # Auth API endpoints
│   ├── todoEndpoints.ts                     # Todo API endpoints
│   └── userEndpoints.ts                     # User API endpoints
└── types/
    ├── apiResponse.types.ts                 # Standard response types
    └── apiError.types.ts                    # Error types
```

### Type Definitions Organization

```
src/shared/types/
├── api.types.ts                             # API-related types
├── common.types.ts                          # Common utility types
├── entities/                                # Domain entity types
│   ├── user.types.ts
│   ├── todo.types.ts
│   └── category.types.ts
├── dto/                                     # Data transfer objects
│   ├── requests/
│   │   ├── createTodo.dto.ts
│   │   └── updateTodo.dto.ts
│   └── responses/
│       ├── todoResponse.dto.ts
│       └── todoSummary.dto.ts
└── enums/
    ├── todoPriority.enum.ts
    └── userRole.enum.ts
```

### Shared Form Components

```
src/shared/components/forms/
├── FormField/
│   ├── FormField.tsx                        # Wrapper with label/error
│   ├── FormField.test.tsx
│   └── index.ts
├── FormSection/
│   ├── FormSection.tsx                      # Section with heading
│   ├── FormSection.test.tsx
│   └── index.ts
├── FormGroup/
│   ├── FormGroup.tsx                        # Group related fields
│   └── index.ts
├── inputs/
│   ├── TextInput/
│   ├── NumberInput/
│   ├── DateInput/
│   ├── SelectInput/
│   ├── CheckboxInput/
│   ├── RadioInput/
│   └── FileInput/
└── validation/
    ├── useFieldValidation.ts
    └── ValidationMessage.tsx
```

---

## Scaling Strategies

### Feature Scaling: When to Split Features

**Indicators for splitting a feature module:**

1. **File count exceeds 30-40 files** in a single feature directory
2. **Multiple distinct subdomains** within the feature
3. **Different team members** working on different aspects
4. **Independent deployment** would be beneficial
5. **ViewModel layer becomes too complex** (5+ ViewModels per feature)

**Example: Splitting a large "todos" feature:**

```
Before:
src/features/todos/                          # Too large, multiple concerns
├── components/
├── viewmodels/                              # 8+ ViewModels
├── models/
└── pages/

After:
src/features/
├── todo-management/                         # Core todo CRUD
│   ├── components/
│   ├── viewmodels/
│   │   ├── useTodoListViewModel.ts
│   │   └── useTodoFormViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
├── todo-categories/                         # Category management
│   ├── components/
│   ├── viewmodels/
│   │   └── useCategoryViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
├── todo-comments/                           # Comments and discussions
│   ├── components/
│   ├── viewmodels/
│   │   └── useCommentsViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
└── todo-analytics/                          # Reporting and analytics
    ├── components/
    ├── viewmodels/
    │   ├── useTodoStatsViewModel.ts
    │   └── useTodoReportsViewModel.ts
    ├── models/
    ├── pages/
    └── services/
```

### Component Library Extraction

When shared components reach critical mass, they should be extracted to a separate package:

```
packages/
├── ui-components/                           # Design system/component library
│   ├── src/
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/                                 # Main backend application
└── frontend/                                # Main frontend application
    └── package.json                         # References ui-components
```

### Domain-Driven Design (DDD) Structure

For very large applications, organization by bounded context is recommended:

```
backend/src/main/java/com/example/todos/
├── shared/                                  # Shared kernel
│   ├── domain/
│   ├── infrastructure/
│   └── application/
├── usermanagement/                          # Bounded context
│   ├── domain/
│   │   ├── model/
│   │   ├── repository/
│   │   └── service/
│   ├── application/
│   │   ├── dto/
│   │   └── service/
│   └── infrastructure/
│       ├── persistence/
│       └── api/
└── todomanagement/                          # Bounded context
    ├── domain/
    ├── application/
    └── infrastructure/
```

---

## Module Organization Patterns

### Backend: Package by Layer vs Package by Feature

**Package by Layer (Not Recommended):**

```
com/example/todos/
├── controller/
│   ├── TodoController.java
│   ├── UserController.java
│   └── CategoryController.java
├── service/
│   ├── TodoService.java
│   ├── UserService.java
│   └── CategoryService.java
├── repository/
│   ├── TodoRepository.java
│   ├── UserRepository.java
│   └── CategoryRepository.java
└── entity/
    ├── TodoEntity.java
    ├── UserEntity.java
    └── CategoryEntity.java
```

**Package by Feature (Recommended):**

```
com/example/todos/
├── todo/
│   ├── TodoController.java
│   ├── TodoService.java
│   ├── TodoRepository.java
│   └── TodoEntity.java
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   └── UserEntity.java
└── category/
    ├── CategoryController.java
    ├── CategoryService.java
    ├── CategoryRepository.java
    └── CategoryEntity.java
```

### Frontend: Component Organization Patterns

**Flat Components (Anti-Pattern):**

```
src/components/
├── TodoList.tsx
├── TodoItem.tsx
├── TodoForm.tsx
├── TodoFilters.tsx
├── UserProfile.tsx
├── UserSettings.tsx
└── ... (100+ components)
```

**Feature-Based with Shared (Recommended):**

```
src/
├── features/
│   ├── todos/
│   │   └── components/
│   │       ├── TodoList/
│   │       ├── TodoItem/
│   │       └── TodoForm/
│   └── users/
│       └── components/
│           ├── UserProfile/
│           └── UserSettings/
└── shared/
    └── components/
        ├── Button/
        └── Input/
```

### Absolute Imports Configuration

Absolute imports should be configured to avoid deep relative paths:

**TypeScript configuration (tsconfig.json):**

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@features/*": ["features/*"],
      "@shared/*": ["shared/*"],
      "@app/*": ["app/*"],
      "@assets/*": ["assets/*"],
      "@types/*": ["types/*"],
      "@utils/*": ["shared/utils/*"],
      "@components/*": ["shared/components/*"],
      "@hooks/*": ["shared/hooks/*"]
    }
  }
}
```

**Usage:**

```typescript
// Instead of:
import { Button } from '../../../shared/components/Button';
import { useAuth } from '../../../../features/auth/hooks/useAuth';

// Use:
import { Button } from '@components/Button';
import { useAuth } from '@features/auth/hooks/useAuth';
```

---

## Build and Deployment Structure

### Environment Configuration

**Backend:**

```
backend/src/main/resources/
├── application.yml                          # Base configuration
├── application-dev.yml                      # Development overrides
├── application-staging.yml                  # Staging overrides
├── application-prod.yml                     # Production overrides
└── application-test.yml                     # Test overrides
```

**Frontend:**

```
frontend/
├── .env                                     # Base environment (committed)
├── .env.local                               # Local overrides (gitignored)
├── .env.development                         # Development environment
├── .env.staging                             # Staging environment
├── .env.production                          # Production environment
└── .env.test                                # Test environment
```

**.env.example:**

```
# API Configuration
VITE_API_BASE_URL=http://localhost:8080/api
VITE_API_TIMEOUT=30000

# Authentication
VITE_AUTH_TOKEN_KEY=auth_token
VITE_REFRESH_TOKEN_KEY=refresh_token

# Feature Flags
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_DEBUG=true
```

### Docker Configuration

```
project-root/
├── docker/
│   ├── backend/
│   │   ├── Dockerfile
│   │   └── Dockerfile.dev
│   ├── frontend/
│   │   ├── Dockerfile
│   │   ├── Dockerfile.dev
│   │   └── nginx.conf
│   └── postgres/
│       └── init.sql
├── docker-compose.yml                       # Development environment
├── docker-compose.prod.yml                  # Production configuration
└── docker-compose.test.yml                  # Testing environment
```

### CI/CD Structure

```
.github/
└── workflows/
    ├── backend-ci.yml                       # Backend CI pipeline
    ├── frontend-ci.yml                      # Frontend CI pipeline
    ├── integration-tests.yml                # Integration tests
    ├── deploy-staging.yml                   # Staging deployment
    └── deploy-production.yml                # Production deployment
```

### Build Artifacts

```
backend/
└── build/                                   # Gradle build output
    ├── classes/
    ├── resources/
    ├── libs/
    │   ├── todos-1.0.0.jar
    │   └── todos-1.0.0-sources.jar
    ├── reports/
    │   ├── tests/
    │   └── jacoco/
    └── test-results/

frontend/
└── dist/                                    # Vite build output
    ├── assets/
    │   ├── index.[hash].js
    │   ├── index.[hash].css
    │   └── vendor.[hash].js
    └── index.html
```

---

## Common Anti-Patterns

### 1. Deep Nesting

**Problem:**

```
src/features/todos/components/forms/complex/subforms/nested/parts/fields/inputs/
└── TodoTitleInput.tsx                       # 10 levels deep!
```

**Solution:**

```
src/features/todos/components/
├── TodoForm/
│   ├── TodoForm.tsx
│   └── subforms/
│       └── TodoDetailsSubform.tsx
└── TodoTitleInput/
    └── TodoTitleInput.tsx                   # Flatten structure
```

### 2. Circular Dependencies

**Problem:**

```
// features/todos/services/todoService.ts
import { userService } from '@features/users/services/userService';

// features/users/services/userService.ts
import { todoService } from '@features/todos/services/todoService';
```

**Solution:**

Extract shared logic to a separate service:

```
// shared/services/todoUserService.ts
export class TodoUserService {
  constructor(
    private todoService: TodoService,
    private userService: UserService
  ) {}
}
```

### 3. God Folders

**Problem:**

```
src/shared/components/                       # 150+ components
├── Component1.tsx
├── Component2.tsx
├── ... (many more)
└── Component150.tsx
```

**Solution:**

Categorize and organize:

```
src/shared/components/
├── forms/                                   # Form-related components
│   ├── Input/
│   └── Select/
├── layout/                                  # Layout components
│   ├── Header/
│   └── Sidebar/
└── feedback/                                # Feedback components
    ├── Alert/
    └── Toast/
```

### 4. Inconsistent Naming

**Problem:**

```
src/features/
├── TodoManagement/                          # PascalCase
├── user-profile/                            # kebab-case
└── shopping_cart/                           # snake_case
```

**Solution:**

Use kebab-case for directories consistently:

```
src/features/
├── todo-management/                         # kebab-case for directories
├── user-profile/
└── shopping-cart/
```

### 5. Missing Index Files

**Problem:**

```
// In TodoList.tsx
import { TodoItem } from './TodoItem/TodoItem';
import { TodoFilters } from './TodoFilters/TodoFilters';
```

**Solution:**

Add index.ts files for cleaner imports:

```
// TodoItem/index.ts
export { TodoItem } from './TodoItem';

// In TodoList.tsx
import { TodoItem } from './TodoItem';
import { TodoFilters } from './TodoFilters';
```

### 6. Mixing Concerns in Feature Modules

**Problem:**

```
src/features/todos/
├── components/
│   ├── TodoList.tsx
│   └── UserAvatar.tsx                       # User concern, not todo
└── services/
    └── authService.ts                       # Auth concern, not todo
```

**Solution:**

Move to appropriate locations:

```
src/features/todos/
├── components/
│   └── TodoList.tsx

src/features/users/
└── components/
    └── UserAvatar.tsx

src/features/auth/
└── services/
    └── authService.ts
```

### 7. Duplication Instead of Abstraction

**Problem:**

```
src/features/todos/utils/dateUtils.ts
src/features/users/utils/dateUtils.ts
src/features/projects/utils/dateUtils.ts     # Same code repeated
```

**Solution:**

Move to shared utilities:

```
src/shared/utils/formatting/dateFormatters.ts
```

### 8. Business Logic in View Components (MVVM Violation)

**Problem:**

```typescript
// Component contains business logic
export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(false);
  
  const loadTodos = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/todos');
      const data = await response.json();
      
      // Business logic in View
      const filtered = data.filter(t => !t.deleted);
      const sorted = filtered.sort((a, b) => 
        a.priority === 'high' ? -1 : 1
      );
      
      setTodos(sorted);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  return <div>{/* Render todos */}</div>;
}
```

**Solution:**

Extract to ViewModel:

```typescript
// viewmodels/useTodoListViewModel.ts
export function useTodoListViewModel() {
  const dispatch = useAppDispatch();
  const todos = useAppSelector(selectFilteredTodos);
  const loading = useAppSelector(selectTodosLoading);
  
  const sortedTodos = useMemo(() => {
    return [...todos].sort((a, b) => 
      a.priority === 'high' ? -1 : 1
    );
  }, [todos]);
  
  const loadTodos = useCallback(() => {
    dispatch(fetchTodos());
  }, [dispatch]);
  
  return { todos: sortedTodos, loading, loadTodos };
}

// Component uses ViewModel
export function TodoList() {
  const viewModel = useTodoListViewModel();
  
  return <div>{/* Render viewModel.todos */}</div>;
}
```

### 9. ViewModels Directly Manipulating DOM

**Problem:**

```typescript
// ViewModel contains DOM manipulation
export function useTodoFormViewModel() {
  const submitForm = () => {
    document.getElementById('submit-btn').classList.add('loading');
    // Submit logic
  };
  
  return { submitForm };
}
```

**Solution:**

Return state, let View handle DOM:

```typescript
// ViewModel returns state
export function useTodoFormViewModel() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const submitForm = async () => {
    setIsSubmitting(true);
    // Submit logic
    setIsSubmitting(false);
  };
  
  return { isSubmitting, submitForm };
}

// View uses state for DOM updates
export function TodoForm() {
  const { isSubmitting, submitForm } = useTodoFormViewModel();
  
  return (
    <button 
      className={isSubmitting ? 'loading' : ''}
      onClick={submitForm}
    >
      Submit
    </button>
  );
}
```

---

## Summary

Effective directory structure for full-stack applications requires:

1. **Feature-based organization** for business logic
2. **MVVM architecture** in the frontend with clear separation of concerns:
   - Models (services, store, validation)
   - ViewModels (custom hooks managing presentation logic)
   - Views (React components)
3. **Colocation** of related files (components, tests, styles, ViewModels)
4. **Clear separation** between feature-specific and shared code
5. **Scalable patterns** that accommodate growth
6. **Consistent naming** and structural conventions
7. **Comprehensive testing** at all levels (including ViewModel layer)
8. **Environment-specific** configuration management
9. **Modern tooling:** Gradle with Kotlin DSL for backend builds, Yarn for frontend package management, Vite for bundling

This structure makes code discoverable, maintainable, and enables teams to work independently on different features without frequent conflicts. As applications grow, teams should regularly evaluate whether the current structure still serves their needs and refactor when necessary.

Key decision points:
- A monorepo simplifies coordination between frontend and backend
- Packaging by feature rather than by layer improves cohesion
- MVVM implementation uses custom hooks as ViewModels
- Shared components should be extracted to separate packages when they reach critical mass
- Nesting should be kept shallow and boundaries clear between features

The goal is not perfect structure from day one, but rather a structure that evolves with the application while maintaining clarity and organization. The MVVM pattern provides clear guidelines for where different types of logic belong, reducing ambiguity and improving testability.
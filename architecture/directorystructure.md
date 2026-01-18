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

Directory structure significantly impacts code maintainability, team collaboration, and application scalability. A well-organized structure enables developers to locate code quickly, understand system architecture, and modify features without affecting unrelated code.

This guide addresses structure for large-scale applications with:
- Multiple features and bounded contexts
- Complex forms with nested subforms
- Numerous pages and routes
- Comprehensive test coverage
- Shared components and utilities
- Multiple deployment environments

### Technology Stack

This guide is tailored for RAMP's tech stack:
- **Backend:** Java with Spring Boot, Gradle build system, PostgreSQL database
- **Frontend:** React with TypeScript, MVVM architecture pattern, Yarn package manager
- **Testing:** JUnit/Mockito (backend), Jest/React Testing Library (frontend)

### Key Principles

**Separation by Feature:** Group related code by feature or domain rather than technical layer.

**MVVM Architecture:** Frontend follows Model-View-ViewModel pattern with custom hooks serving as ViewModels.

**Colocation:** Place related files close together. Tests, types, and styles should live near the code they support.

**Discoverability:** Structure should make it obvious where new code belongs and where existing code lives.

**Scalability:** Organization should accommodate growth without requiring major restructuring.

---

## Repository Organization

### Option 1: Monorepo (Recommended for Most Projects)

Single repository containing both frontend and backend.

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

**Advantages:**
- Simplified dependency management
- Atomic commits across stack
- Easier code sharing and refactoring
- Single CI/CD pipeline
- Unified versioning

**Disadvantages:**
- Larger repository size
- Potential for tighter coupling
- Build complexity

### Option 2: Multi-Repo (Separate Repositories)

Separate repositories for frontend and backend.

```
backend-repo/
├── src/
├── pom.xml
└── README.md

frontend-repo/
├── src/
├── package.json
└── README.md
```

**Advantages:**
- Independent deployment cycles
- Clearer ownership boundaries
- Smaller repository size
- Language-specific tooling

**Disadvantages:**
- Coordination overhead for breaking changes
- Duplicate configuration
- Version synchronization complexity

**Recommendation:** Use monorepo unless teams are organizationally separate or deployment independence is critical.

---

## Backend Structure (Java/Spring)

### Standard Gradle Project Layout

```
backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── company/
│   │   │           └── projectname/
│   │   │               ├── ProjectApplication.java
│   │   │               ├── config/              # Application configuration
│   │   │               │   ├── SecurityConfig.java
│   │   │               │   ├── DatabaseConfig.java
│   │   │               │   ├── CorsConfig.java
│   │   │               │   └── SwaggerConfig.java
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
│   │   │               ├── feature1/            # Feature-based modules
│   │   │               │   ├── controller/
│   │   │               │   │   └── Feature1Controller.java
│   │   │               │   ├── dto/
│   │   │               │   │   ├── Feature1Request.java
│   │   │               │   │   ├── Feature1Response.java
│   │   │               │   │   └── Feature1Summary.java
│   │   │               │   ├── entity/
│   │   │               │   │   └── Feature1Entity.java
│   │   │               │   ├── repository/
│   │   │               │   │   └── Feature1Repository.java
│   │   │               │   ├── service/
│   │   │               │   │   ├── Feature1Service.java
│   │   │               │   │   └── Feature1ValidationService.java
│   │   │               │   └── mapper/
│   │   │               │       └── Feature1Mapper.java
│   │   │               ├── feature2/
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
│   │       │   └── migration/               # Flyway/Liquibase migrations
│   │       │       ├── V1__initial_schema.sql
│   │       │       ├── V2__add_users_table.sql
│   │       │       └── V3__add_feature1_tables.sql
│   │       ├── static/                      # Static resources (if any)
│   │       └── templates/                   # Email templates, etc.
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── company/
│       │           └── projectname/
│       │               ├── feature1/
│       │               │   ├── controller/
│       │               │   │   └── Feature1ControllerTest.java
│       │               │   ├── service/
│       │               │   │   └── Feature1ServiceTest.java
│       │               │   ├── repository/
│       │               │   │   └── Feature1RepositoryTest.java
│       │               │   └── integration/
│       │               │       └── Feature1IntegrationTest.java
│       │               └── common/
│       │                   └── BaseIntegrationTest.java
│       └── resources/
│           ├── application-test.yml
│           └── test-data/
│               └── feature1-test-data.sql
├── build/                                   # Build output (gitignored)
│   ├── classes/
│   ├── libs/
│   └── reports/
├── gradle/                                  # Gradle wrapper
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── build.gradle                             # Gradle build configuration
├── settings.gradle                          # Gradle settings
├── gradlew                                  # Gradle wrapper script (Unix)
├── gradlew.bat                              # Gradle wrapper script (Windows)
├── .gitignore
└── README.md
```

### Feature Module Deep Dive

For complex features with multiple subdomains:

```
backend/src/main/java/com/company/projectname/
└── taskmanagement/                          # Feature module
    ├── controller/
    │   ├── TaskController.java              # Main task operations
    │   ├── TaskCategoryController.java      # Category management
    │   └── TaskCommentController.java       # Comments on tasks
    ├── dto/
    │   ├── task/
    │   │   ├── CreateTaskRequest.java
    │   │   ├── UpdateTaskRequest.java
    │   │   ├── TaskResponse.java
    │   │   └── TaskSummary.java
    │   ├── category/
    │   │   ├── CategoryRequest.java
    │   │   └── CategoryResponse.java
    │   └── comment/
    │       ├── CommentRequest.java
    │       └── CommentResponse.java
    ├── entity/
    │   ├── TaskEntity.java
    │   ├── CategoryEntity.java
    │   ├── CommentEntity.java
    │   └── TaskPriority.java               # Enums
    ├── repository/
    │   ├── TaskRepository.java
    │   ├── CategoryRepository.java
    │   ├── CommentRepository.java
    │   └── specification/                   # Custom queries
    │       └── TaskSpecification.java
    ├── service/
    │   ├── TaskService.java
    │   ├── CategoryService.java
    │   ├── CommentService.java
    │   └── impl/                            # Service implementations if using interfaces
    │       ├── TaskServiceImpl.java
    │       ├── CategoryServiceImpl.java
    │       └── CommentServiceImpl.java
    └── mapper/
        ├── TaskMapper.java
        ├── CategoryMapper.java
        └── CommentMapper.java
```

### Configuration Organization

```
backend/src/main/java/com/company/projectname/config/
├── SecurityConfig.java                      # Spring Security configuration
├── WebConfig.java                           # Web MVC configuration
├── DatabaseConfig.java                      # DataSource and JPA configuration
├── CacheConfig.java                         # Redis/Caffeine cache configuration
├── AsyncConfig.java                         # Async execution configuration
├── SwaggerConfig.java                       # API documentation
├── CorsConfig.java                          # CORS policy
└── properties/                              # Type-safe configuration properties
    ├── AppProperties.java
    ├── JwtProperties.java
    └── StorageProperties.java
```

### Gradle Configuration Example

**build.gradle:**

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.company'
version = '1.0.0'
sourceCompatibility = '21'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot starters
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    
    // Database
    runtimeOnly 'org.postgresql:postgresql'
    implementation 'org.flywaydb:flyway-core'
    
    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    // MapStruct
    implementation 'org.mapstruct:mapstruct:1.5.5.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'
    
    // Testing
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testImplementation 'org.testcontainers:postgresql'
    testImplementation 'org.testcontainers:junit-jupiter'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

**settings.gradle:**

```gradle
rootProject.name = 'projectname'
```

---

## Frontend Structure (React/TypeScript)

### Feature-Based Organization with MVVM (Recommended for Large Apps)

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
│   ├── index.tsx                            # Entry point
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
│   │   ├── tasks/                           # Task management feature
│   │   │   ├── components/                  # View layer
│   │   │   │   ├── TaskList/
│   │   │   │   │   ├── TaskList.tsx
│   │   │   │   │   ├── TaskList.test.tsx
│   │   │   │   │   ├── TaskList.module.css
│   │   │   │   │   └── index.ts
│   │   │   │   ├── TaskItem/
│   │   │   │   ├── TaskForm/
│   │   │   │   │   ├── TaskForm.tsx
│   │   │   │   │   ├── TaskForm.test.tsx
│   │   │   │   │   ├── subforms/           # Nested subforms
│   │   │   │   │   │   ├── TaskDetailsSubform.tsx
│   │   │   │   │   │   ├── TaskCategorySubform.tsx
│   │   │   │   │   │   └── TaskAttachmentsSubform.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   ├── TaskFilters/
│   │   │   │   └── TaskStats/
│   │   │   ├── pages/                       # View layer (page components)
│   │   │   │   ├── TaskListPage.tsx
│   │   │   │   ├── TaskDetailPage.tsx
│   │   │   │   └── TaskEditPage.tsx
│   │   │   ├── viewmodels/                  # ViewModel layer
│   │   │   │   ├── useTaskListViewModel.ts  # List ViewModel
│   │   │   │   ├── useTaskFormViewModel.ts  # Form ViewModel
│   │   │   │   ├── useTaskDetailViewModel.ts # Detail ViewModel
│   │   │   │   └── useTaskFiltersViewModel.ts # Filters ViewModel
│   │   │   ├── hooks/                       # Additional custom hooks
│   │   │   │   ├── useTaskPagination.ts
│   │   │   │   └── useTaskSort.ts
│   │   │   ├── models/                      # Model layer
│   │   │   │   ├── services/
│   │   │   │   │   ├── taskService.ts
│   │   │   │   │   └── categoryService.ts
│   │   │   │   ├── store/
│   │   │   │   │   ├── taskSlice.ts
│   │   │   │   │   ├── taskSelectors.ts
│   │   │   │   │   └── taskThunks.ts
│   │   │   │   └── validation/
│   │   │   │       └── taskValidation.ts
│   │   │   ├── types/
│   │   │   │   ├── task.types.ts
│   │   │   │   ├── category.types.ts
│   │   │   │   └── api.types.ts
│   │   │   ├── utils/
│   │   │   │   └── taskHelpers.ts
│   │   │   └── index.ts
│   │   ├── dashboard/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── viewmodels/
│   │   │   │   └── useDashboardViewModel.ts
│   │   │   ├── models/
│   │   │   └── widgets/                     # Dashboard-specific widgets
│   │   │       ├── RecentTasksWidget/
│   │   │       ├── TaskStatsWidget/
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
├── vite.config.ts                           # or webpack.config.js
├── .eslintrc.json
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
// src/features/tasks/viewmodels/useTaskListViewModel.ts
import { useState, useEffect, useMemo, useCallback } from 'react';
import { useAppDispatch, useAppSelector } from '@app/store/hooks';
import { fetchTasks, selectFilteredTasks, selectTasksLoading } from '../models/store/taskSlice';
import { taskService } from '../models/services/taskService';
import type { Task, TaskFilter } from '../types/task.types';

export function useTaskListViewModel() {
  // State from Model layer
  const dispatch = useAppDispatch();
  const tasks = useAppSelector(selectFilteredTasks);
  const loading = useAppSelector(selectTasksLoading);
  
  // ViewModel-specific state
  const [filter, setFilter] = useState<TaskFilter>('all');
  const [searchTerm, setSearchTerm] = useState('');
  
  // Computed properties for View
  const filteredTasks = useMemo(() => {
    return tasks.filter(task => 
      task.title.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [tasks, searchTerm]);
  
  const activeCount = useMemo(() => {
    return tasks.filter(t => !t.completed).length;
  }, [tasks]);
  
  const completedCount = useMemo(() => {
    return tasks.filter(t => t.completed).length;
  }, [tasks]);
  
  // Commands for View to invoke
  const loadTasks = useCallback(() => {
    dispatch(fetchTasks());
  }, [dispatch]);
  
  const handleFilterChange = useCallback((newFilter: TaskFilter) => {
    setFilter(newFilter);
  }, []);
  
  const handleSearchChange = useCallback((term: string) => {
    setSearchTerm(term);
  }, []);
  
  const deleteTask = useCallback(async (taskId: string) => {
    await taskService.deleteTask(taskId);
    dispatch(fetchTasks());
  }, [dispatch]);
  
  // Load data on mount
  useEffect(() => {
    loadTasks();
  }, [loadTasks]);
  
  // Return ViewModel interface
  return {
    // State
    tasks: filteredTasks,
    loading,
    filter,
    searchTerm,
    activeCount,
    completedCount,
    
    // Commands
    loadTasks,
    handleFilterChange,
    handleSearchChange,
    deleteTask,
  };
}
```

**Example View Using ViewModel:**

```typescript
// src/features/tasks/components/TaskList/TaskList.tsx
import { useTaskListViewModel } from '../../viewmodels/useTaskListViewModel';

export function TaskList() {
  const viewModel = useTaskListViewModel();
  
  if (viewModel.loading) {
    return <LoadingSpinner />;
  }
  
  return (
    <div>
      <input
        type="text"
        value={viewModel.searchTerm}
        onChange={(e) => viewModel.handleSearchChange(e.target.value)}
        placeholder="Search tasks..."
      />
      
      <div>
        <button onClick={() => viewModel.handleFilterChange('all')}>
          All ({viewModel.tasks.length})
        </button>
        <button onClick={() => viewModel.handleFilterChange('active')}>
          Active ({viewModel.activeCount})
        </button>
        <button onClick={() => viewModel.handleFilterChange('completed')}>
          Completed ({viewModel.completedCount})
        </button>
      </div>
      
      <ul>
        {viewModel.tasks.map(task => (
          <TaskItem
            key={task.id}
            task={task}
            onDelete={viewModel.deleteTask}
          />
        ))}
      </ul>
    </div>
  );
}
```

### Page Organization for Large Applications

```
src/features/tasks/pages/
├── TaskListPage/
│   ├── TaskListPage.tsx
│   ├── TaskListPage.test.tsx
│   ├── components/                          # Page-specific components
│   │   ├── TaskListHeader.tsx
│   │   ├── TaskListToolbar.tsx
│   │   └── EmptyTasksPlaceholder.tsx
│   ├── hooks/
│   │   ├── useTaskListPage.ts               # Page-specific logic
│   │   └── useTaskListFilters.ts
│   └── index.ts
├── TaskDetailPage/
│   ├── TaskDetailPage.tsx
│   ├── TaskDetailPage.test.tsx
│   ├── components/
│   │   ├── TaskHeader.tsx
│   │   ├── TaskMetadata.tsx
│   │   ├── TaskComments.tsx
│   │   └── TaskActivityLog.tsx
│   ├── hooks/
│   │   └── useTaskDetailPage.ts
│   └── index.ts
└── TaskEditPage/
    ├── TaskEditPage.tsx
    ├── TaskEditPage.test.tsx
    └── index.ts
```

### Frontend Package Configuration

**package.json (with Yarn):**

```json
{
  "name": "projectname-frontend",
  "version": "1.0.0",
  "packageManager": "yarn@4.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:viewmodels": "jest --testPathPattern=viewmodels",
    "test:components": "jest --testPathPattern=components",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "playwright test",
    "lint": "eslint src --ext ts,tsx",
    "lint:fix": "eslint src --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "@reduxjs/toolkit": "^2.0.0",
    "react-redux": "^9.0.0",
    "axios": "^1.6.0",
    "zod": "^3.22.0",
    "date-fns": "^3.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@types/jest": "^29.5.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "@testing-library/react-hooks": "^8.0.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "jest": "^29.7.0",
    "jest-environment-jsdom": "^29.7.0",
    "eslint": "^8.55.0",
    "prettier": "^3.1.0",
    "msw": "^2.0.0",
    "@playwright/test": "^1.40.0"
  }
}
```

---

## Testing Organization

### Backend Testing Structure

```
backend/src/test/java/com/company/projectname/
├── feature1/
│   ├── controller/
│   │   └── Feature1ControllerTest.java      # Controller layer tests
│   ├── service/
│   │   └── Feature1ServiceTest.java         # Service layer unit tests
│   ├── repository/
│   │   └── Feature1RepositoryTest.java      # Repository tests
│   ├── mapper/
│   │   └── Feature1MapperTest.java          # Mapper tests
│   └── integration/
│       ├── Feature1IntegrationTest.java     # End-to-end feature tests
│       └── Feature1ApiTest.java             # API contract tests
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
│   ├── tasks.sql
│   └── categories.sql
├── fixtures/                                # JSON test fixtures
│   ├── valid-task-request.json
│   └── invalid-task-request.json
└── contracts/                               # API contract definitions
    └── task-api-contract.json
```

### Frontend Testing Structure

```
src/features/tasks/
├── components/
│   ├── TaskList/
│   │   ├── TaskList.tsx
│   │   ├── TaskList.test.tsx                # Component unit tests (View)
│   │   └── TaskList.integration.test.tsx   # Component integration tests
│   └── TaskForm/
│       ├── TaskForm.tsx
│       ├── TaskForm.test.tsx
│       └── subforms/
│           └── TaskDetailsSubform/
│               ├── TaskDetailsSubform.tsx
│               └── TaskDetailsSubform.test.tsx
├── viewmodels/
│   ├── useTaskListViewModel.ts
│   ├── useTaskListViewModel.test.ts         # ViewModel tests
│   ├── useTaskFormViewModel.ts
│   └── useTaskFormViewModel.test.ts
├── hooks/
│   ├── useTaskPagination.ts
│   └── useTaskPagination.test.ts            # Hook tests
├── models/
│   ├── services/
│   │   ├── taskService.ts
│   │   └── taskService.test.ts              # Service tests (Model)
│   ├── store/
│   │   ├── taskSlice.ts
│   │   └── taskSlice.test.ts                # Redux slice tests (Model)
│   └── validation/
│       ├── taskValidation.ts
│       └── taskValidation.test.ts           # Validation tests (Model)
└── __tests__/                               # Feature-level tests
    ├── tasks.integration.test.tsx           # Feature integration tests
    └── tasks.e2e.test.tsx                   # E2E tests (or in separate e2e/ directory)
```

**Global Test Setup:**

```
tests/
├── setup.ts                                 # Global test setup
├── mocks/
│   ├── handlers/                            # MSW request handlers
│   │   ├── authHandlers.ts
│   │   ├── taskHandlers.ts
│   │   └── index.ts
│   └── server.ts                            # MSW server setup
├── utils/
│   ├── testUtils.tsx                        # Custom render functions
│   ├── renderWithProviders.tsx              # Render with Redux/Router
│   ├── mockViewModels.ts                    # Mock ViewModel factories
│   └── mockData.ts                          # Mock data generators
└── fixtures/
    ├── tasks.ts                             # Test data fixtures
    └── users.ts
```

**ViewModel Testing Example:**

```typescript
// src/features/tasks/viewmodels/useTaskListViewModel.test.ts
import { renderHook, act, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { useTaskListViewModel } from './useTaskListViewModel';
import { taskSlice } from '../models/store/taskSlice';
import * as taskService from '../models/services/taskService';

jest.mock('../models/services/taskService');

describe('useTaskListViewModel', () => {
  let store;
  
  beforeEach(() => {
    store = configureStore({
      reducer: { tasks: taskSlice.reducer }
    });
  });
  
  it('loads tasks on mount', async () => {
    const mockTasks = [
      { id: '1', title: 'Task 1', completed: false },
      { id: '2', title: 'Task 2', completed: true }
    ];
    
    (taskService.getTasks as jest.Mock).mockResolvedValue(mockTasks);
    
    const wrapper = ({ children }) => (
      <Provider store={store}>{children}</Provider>
    );
    
    const { result } = renderHook(() => useTaskListViewModel(), { wrapper });
    
    await waitFor(() => {
      expect(result.current.tasks).toHaveLength(2);
    });
  });
  
  it('filters tasks by search term', () => {
    const wrapper = ({ children }) => (
      <Provider store={store}>{children}</Provider>
    );
    
    const { result } = renderHook(() => useTaskListViewModel(), { wrapper });
    
    act(() => {
      result.current.handleSearchChange('Task 1');
    });
    
    expect(result.current.tasks).toHaveLength(1);
    expect(result.current.tasks[0].title).toBe('Task 1');
  });
  
  it('calculates active and completed counts correctly', () => {
    // Test computed properties
  });
});
```

**E2E Testing Structure (Cypress/Playwright):**

```
frontend/
└── e2e/                                     # Or cypress/ for Cypress
    ├── fixtures/
    │   ├── tasks.json
    │   └── users.json
    ├── support/
    │   ├── commands.ts                      # Custom commands
    │   └── index.ts
    └── tests/
        ├── auth/
        │   ├── login.spec.ts
        │   └── registration.spec.ts
        ├── tasks/
        │   ├── task-creation.spec.ts
        │   ├── task-editing.spec.ts
        │   └── task-filtering.spec.ts
        └── workflows/
            └── complete-task-workflow.spec.ts
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
│   ├── taskEndpoints.ts                     # Task API endpoints
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
│   ├── task.types.ts
│   └── category.types.ts
├── dto/                                     # Data transfer objects
│   ├── requests/
│   │   ├── createTask.dto.ts
│   │   └── updateTask.dto.ts
│   └── responses/
│       ├── taskResponse.dto.ts
│       └── taskSummary.dto.ts
└── enums/
    ├── taskPriority.enum.ts
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
3. **Different teams** working on different aspects
4. **Independent deployment** would be beneficial
5. **ViewModel layer becomes too complex** (5+ ViewModels per feature)

**Example: Splitting a large "tasks" feature:**

```
Before:
src/features/tasks/                          # Too large, multiple concerns
├── components/
├── viewmodels/                              # 8+ ViewModels
├── models/
└── pages/

After:
src/features/
├── task-management/                         # Core task CRUD
│   ├── components/
│   ├── viewmodels/
│   │   ├── useTaskListViewModel.ts
│   │   └── useTaskFormViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
├── task-categories/                         # Category management
│   ├── components/
│   ├── viewmodels/
│   │   └── useCategoryViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
├── task-comments/                           # Comments and discussions
│   ├── components/
│   ├── viewmodels/
│   │   └── useCommentsViewModel.ts
│   ├── models/
│   ├── pages/
│   └── services/
└── task-analytics/                          # Reporting and analytics
    ├── components/
    ├── viewmodels/
    │   ├── useTaskStatsViewModel.ts
    │   └── useTaskReportsViewModel.ts
    ├── models/
    ├── pages/
    └── services/
```

### Component Library Extraction

When shared components reach critical mass, extract to a separate package:

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

For very large applications, organize by bounded context:

```
backend/src/main/java/com/company/projectname/
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
└── taskmanagement/                          # Bounded context
    ├── domain/
    ├── application/
    └── infrastructure/
```

---

## Module Organization Patterns

### Backend: Package by Layer vs Package by Feature

**Package by Layer (Not Recommended for Large Apps):**

```
com/company/projectname/
├── controller/
│   ├── TaskController.java
│   ├── UserController.java
│   └── CategoryController.java
├── service/
│   ├── TaskService.java
│   ├── UserService.java
│   └── CategoryService.java
├── repository/
│   ├── TaskRepository.java
│   ├── UserRepository.java
│   └── CategoryRepository.java
└── entity/
    ├── TaskEntity.java
    ├── UserEntity.java
    └── CategoryEntity.java
```

**Package by Feature (Recommended):**

```
com/company/projectname/
├── task/
│   ├── TaskController.java
│   ├── TaskService.java
│   ├── TaskRepository.java
│   └── TaskEntity.java
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

**Flat Components (Anti-Pattern for Large Apps):**

```
src/components/
├── TaskList.tsx
├── TaskItem.tsx
├── TaskForm.tsx
├── TaskFilters.tsx
├── UserProfile.tsx
├── UserSettings.tsx
└── ... (100+ components)
```

**Feature-Based with Shared (Recommended):**

```
src/
├── features/
│   ├── tasks/
│   │   └── components/
│   │       ├── TaskList/
│   │       ├── TaskItem/
│   │       └── TaskForm/
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

Configure absolute imports to avoid deep relative paths:

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
    │   ├── projectname-1.0.0.jar
    │   └── projectname-1.0.0-sources.jar
    ├── reports/
    │   ├── tests/
    │   └── jacoco/
    └── test-results/

frontend/
└── dist/                                    # Vite/Webpack build output
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
src/features/tasks/components/forms/complex/subforms/nested/parts/fields/inputs/
└── TaskTitleInput.tsx                       # 10 levels deep!
```

**Solution:**

```
src/features/tasks/components/
├── TaskForm/
│   ├── TaskForm.tsx
│   └── subforms/
│       └── TaskDetailsSubform.tsx
└── TaskTitleInput/
    └── TaskTitleInput.tsx                   # Flatten structure
```

### 2. Circular Dependencies

**Problem:**

```
// features/tasks/services/taskService.ts
import { userService } from '@features/users/services/userService';

// features/users/services/userService.ts
import { taskService } from '@features/tasks/services/taskService';
```

**Solution:**

Extract shared logic to a separate service:

```
// shared/services/taskUserService.ts
export class TaskUserService {
  constructor(
    private taskService: TaskService,
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
├── TaskManagement/                          # PascalCase
├── user-profile/                            # kebab-case
└── shopping_cart/                           # snake_case
```

**Solution:**

Choose one convention and enforce it:

```
src/features/
├── task-management/                         # kebab-case for directories
├── user-profile/
└── shopping-cart/
```

### 5. Missing Index Files

**Problem:**

```
// In TaskList.tsx
import { TaskItem } from './TaskItem/TaskItem';
import { TaskFilters } from './TaskFilters/TaskFilters';
```

**Solution:**

Add index.ts files for cleaner imports:

```
// TaskItem/index.ts
export { TaskItem } from './TaskItem';

// In TaskList.tsx
import { TaskItem } from './TaskItem';
import { TaskFilters } from './TaskFilters';
```

### 6. Mixing Concerns in Feature Modules

**Problem:**

```
src/features/tasks/
├── components/
│   ├── TaskList.tsx
│   └── UserAvatar.tsx                       # User concern, not task
└── services/
    └── authService.ts                       # Auth concern, not task
```

**Solution:**

Move to appropriate locations:

```
src/features/tasks/
├── components/
│   └── TaskList.tsx

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
src/features/tasks/utils/dateUtils.ts
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
export function TaskList() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState(false);
  
  const loadTasks = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/tasks');
      const data = await response.json();
      
      // Business logic in View
      const filtered = data.filter(t => !t.deleted);
      const sorted = filtered.sort((a, b) => 
        a.priority === 'high' ? -1 : 1
      );
      
      setTasks(sorted);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  return <div>{/* Render tasks */}</div>;
}
```

**Solution:**

Extract to ViewModel:

```typescript
// viewmodels/useTaskListViewModel.ts
export function useTaskListViewModel() {
  const dispatch = useAppDispatch();
  const tasks = useAppSelector(selectFilteredTasks);
  const loading = useAppSelector(selectTasksLoading);
  
  const sortedTasks = useMemo(() => {
    return [...tasks].sort((a, b) => 
      a.priority === 'high' ? -1 : 1
    );
  }, [tasks]);
  
  const loadTasks = useCallback(() => {
    dispatch(fetchTasks());
  }, [dispatch]);
  
  return { tasks: sortedTasks, loading, loadTasks };
}

// Component uses ViewModel
export function TaskList() {
  const viewModel = useTaskListViewModel();
  
  return <div>{/* Render viewModel.tasks */}</div>;
}
```

### 9. ViewModels Directly Manipulating DOM

**Problem:**

```typescript
// ViewModel contains DOM manipulation
export function useTaskFormViewModel() {
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
export function useTaskFormViewModel() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const submitForm = async () => {
    setIsSubmitting(true);
    // Submit logic
    setIsSubmitting(false);
  };
  
  return { isSubmitting, submitForm };
}

// View uses state for DOM updates
export function TaskForm() {
  const { isSubmitting, submitForm } = useTaskFormViewModel();
  
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

Effective directory structure for large full-stack applications requires:

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
9. **Modern tooling:** Gradle for backend builds, Yarn for frontend package management

The structure should make code discoverable, maintainable, and enable teams to work independently on different features without frequent conflicts. As applications grow, regularly evaluate whether the current structure still serves the team's needs and refactor when necessary.

Key decision points:
- Monorepo vs multi-repo based on team structure and deployment needs
- Package by feature vs package by layer (prefer feature for large apps)
- MVVM implementation strategy (custom hooks as ViewModels)
- When to extract shared components to separate packages
- How deeply to nest components and modules
- Where to draw boundaries between features

The goal is not perfect structure from day one, but rather a structure that evolves with the application while maintaining clarity and organization. The MVVM pattern provides clear guidelines for where different types of logic belong, reducing ambiguity and improving testability.
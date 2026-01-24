# Controller Layer in Multilayered Java/Spring/Postgres Architecture
# Table of Contents

- [**Architecture Overview**](#architecture-overview)

#### Part 1: The Basics
- [**What is the Controller Layer?**](#what-is-the-controller-layer)
- [**Basic Controller Structure**](#basic-controller-structure)
- [**Key Annotations**](#key-annotations)

#### Part 2: The Complete Layer Stack
- [**Entity Layer**](#entity-layer)
- [**DTO Layer**](#dto-layer)
- [**Mapper Layer**](#mapper-layer)
- [**Repository Layer**](#repository-layer)
- [**Service Layer**](#service-layer)
- [**Controller Layer (Complete)**](#controller-layer-complete)

#### Part 3: Advanced Controller Patterns
- [**Pagination and Sorting**](#pagination-and-sorting)
- [**Filtering and Search**](#filtering-and-search)
- [**Bulk Operations**](#bulk-operations)
- [**Custom Response Wrappers**](#custom-response-wrappers)

#### Part 4: Authorization Annotations
- [**Authorization Annotations**](#part-4-authorization-annotations)

#### Extra: Common Pitfalls
- [**Business Logic in Controllers**](#business-logic-in-controllers)
- [**Returning Entities Instead of DTOs**](#returning-entities-instead-of-dtos)
- [**Not Using Proper HTTP Status Codes**](#not-using-proper-http-status-codes)
- [**Missing @Valid Annotation**](#missing-valid-annotation)
- [**Not Handling Transaction Boundaries Properly**](#not-handling-transaction-boundaries-properly)
- [**Exposing Too Much in Error Messages**](#exposing-too-much-in-error-messages)
- [**Not Using @PathVariable and @RequestParam Correctly**](#not-using-pathvariable-and-requestparam-correctly)

#### Extra: Best Practices
- [**Use Constructor Injection (Lombok)**](#use-constructor-injection-lombok)
- [**Keep Controllers Thin**](#keep-controllers-thin)
- [**Use Proper REST Conventions**](#use-proper-rest-conventions)
- [**Use API Versioning**](#use-api-versioning)
- [**HTTP Status Codes**](#http-status-codes)
- [**Key Annotations Reference**](#key-annotations-reference)
- [**How @Valid Works**](#how-valid-works)

#### Architecture Overview

In a multilayered Spring Boot application, the controller layer is the **presentation layer** that handles HTTP requests and responses. Here's how the layers interact:

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT (React/Angular)                   │
└─────────────────────────────────────────────────────────────┘
                              ↕ HTTP
┌─────────────────────────────────────────────────────────────┐
│  CONTROLLER LAYER                                            │
│  - Receives HTTP requests                                    │
│  - Validates request data                                    │
│  - Calls service layer                                       │
│  - Maps domain objects to DTOs                               │
│  - Returns HTTP responses                                    │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│  SERVICE LAYER                                               │
│  - Business logic                                            │
│  - Transaction management                                    │
│  - Orchestrates operations                                   │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│  REPOSITORY LAYER                                            │
│  - Data access                                               │
│  - Database queries                                          │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│  DATABASE (PostgreSQL)                                       │
│  - Entity tables                                             │
│  - Relationships                                             │
└─────────────────────────────────────────────────────────────┘

Supporting Layers:
- DTOs (Data Transfer Objects): Request/Response models
- Mappers: Convert between Entities and DTOs
- Entities: Domain models (JPA entities)
```

## Part 1: The Basics

### What is the Controller Layer?

The controller layer is responsible for:
- receiving HTTP requests from clients
- validating input data
- delegating business logic to the service layer
- mapping between DTOs and domain entities
- returning appropriate HTTP responses

The controller is NOT responsible for:
- contain business logic
- access repository directly
- expose entities directly
- perform complex data transformations
- manage transactions


### Basic Controller Structure

```java
package com.example.todoapp.controller;

import com.example.todoapp.dto.TodoDTO;
import com.example.todoapp.dto.CreateTodoRequest;
import com.example.todoapp.dto.UpdateTodoRequest;
import com.example.todoapp.service.TodoService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    // GET all todos
    @GetMapping
    public ResponseEntity<List<TodoDTO>> getAllTodos() {
        List<TodoDTO> todos = todoService.getAllTodos();
        return ResponseEntity.ok(todos);
    }
    
    // GET single todo by ID
    @GetMapping("/{id}")
    public ResponseEntity<TodoDTO> getTodoById(@PathVariable Long id) {
        TodoDTO todo = todoService.getTodoById(id);
        return ResponseEntity.ok(todo);
    }
    
    // CREATE new todo
    @PostMapping
    public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
        TodoDTO todo = todoService.createTodo(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(todo);
    }
    
    // UPDATE existing todo
    @PutMapping("/{id}")
    public ResponseEntity<TodoDTO> updateTodo(
            @PathVariable Long id,
            @Valid @RequestBody UpdateTodoRequest request) {
        TodoDTO todo = todoService.updateTodo(id, request);
        return ResponseEntity.ok(todo);
    }
    
    // DELETE todo
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id) {
        todoService.deleteTodo(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Key Annotations

```java
// class-level annotations
@RestController              // combines @Controller + @ResponseBody
@RequestMapping("/api/todos") // Base URL path for all endpoints
@RequiredArgsConstructor     // Lombok: generates constructor for final fields

// method-level annotations
@GetMapping                  // HTTP GET
@PostMapping                 // HTTP POST
@PutMapping                  // HTTP PUT
@PatchMapping               // HTTP PATCH
@DeleteMapping              // HTTP DELETE

// parameter annotations
@PathVariable               // extract value from URL path
@RequestParam               // extract value from query string
@RequestBody                // parse JSON from request body
@Valid                      // enable validation on DTOs
```

## Part 2: The Complete Layer Stack

### Entity Layer

```java
package com.example.todoapp.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "todos")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Todo {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String title;
    
    @Column(length = 500)
    private String description;
    
    @Column(nullable = false)
    private Boolean completed = false;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Priority priority = Priority.MEDIUM;
    
    private LocalDateTime dueDate;
    
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @Column(nullable = false)
    private LocalDateTime updatedAt;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}

enum Priority {
    LOW, MEDIUM, HIGH
}
```

### DTO Layer

```java
package com.example.todoapp.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;

// response DTO - what we send to clients
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TodoDTO {
    private Long id;
    private String title;
    private String description;
    private Boolean completed;
    private String priority;
    private LocalDateTime dueDate;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private Long userId;
}

// request DTO for creating todos
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CreateTodoRequest {
    
    @NotBlank(message = "Title is required")
    @Size(min = 1, max = 100, message = "Title must be between 1 and 100 characters")
    private String title;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @Pattern(regexp = "LOW|MEDIUM|HIGH", message = "Priority must be LOW, MEDIUM, or HIGH")
    private String priority = "MEDIUM";
    
    private LocalDateTime dueDate;
}

// request DTO for updating todos
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UpdateTodoRequest {
    
    @Size(min = 1, max = 100, message = "Title must be between 1 and 100 characters")
    private String title;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    private Boolean completed;
    
    @Pattern(regexp = "LOW|MEDIUM|HIGH", message = "Priority must be LOW, MEDIUM, or HIGH")
    private String priority;
    
    private LocalDateTime dueDate;
}
```

### Mapper Layer

```java
package com.example.todoapp.mapper;

import com.example.todoapp.dto.TodoDTO;
import com.example.todoapp.dto.CreateTodoRequest;
import com.example.todoapp.entity.Todo;
import com.example.todoapp.entity.Priority;
import org.springframework.stereotype.Component;

@Component
public class TodoMapper {
    
    // entity -> dto
    public TodoDTO toDTO(Todo todo) {
        if (todo == null) return null;
        
        TodoDTO dto = new TodoDTO();
        dto.setId(todo.getId());
        dto.setTitle(todo.getTitle());
        dto.setDescription(todo.getDescription());
        dto.setCompleted(todo.getCompleted());
        dto.setPriority(todo.getPriority().name());
        dto.setDueDate(todo.getDueDate());
        dto.setCreatedAt(todo.getCreatedAt());
        dto.setUpdatedAt(todo.getUpdatedAt());
        dto.setUserId(todo.getUser().getId());
        
        return dto;
    }
    
    // CreateRequest -> Entity
    public Todo toEntity(CreateTodoRequest request) {
        if (request == null) return null;
        
        Todo todo = new Todo();
        todo.setTitle(request.getTitle());
        todo.setDescription(request.getDescription());
        todo.setPriority(Priority.valueOf(request.getPriority()));
        todo.setDueDate(request.getDueDate());
        todo.setCompleted(false);
        
        return todo;
    }
    
    // update Entity from UpdateRequest
    public void updateEntityFromDTO(Todo todo, UpdateTodoRequest request) {
        if (request.getTitle() != null) {
            todo.setTitle(request.getTitle());
        }
        if (request.getDescription() != null) {
            todo.setDescription(request.getDescription());
        }
        if (request.getCompleted() != null) {
            todo.setCompleted(request.getCompleted());
        }
        if (request.getPriority() != null) {
            todo.setPriority(Priority.valueOf(request.getPriority()));
        }
        if (request.getDueDate() != null) {
            todo.setDueDate(request.getDueDate());
        }
    }
}
```

### Repository Layer

```java
package com.example.todoapp.repository;

import com.example.todoapp.entity.Todo;
import com.example.todoapp.entity.Priority;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;
import java.time.LocalDateTime;
import java.util.List;

@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {
    
    // find by user ID
    List<Todo> findByUserId(Long userId);
    
    // find completed todos
    List<Todo> findByCompletedTrue();
    
    // find by priority
    List<Todo> findByPriority(Priority priority);
    
    // find overdue todos
    @Query("SELECT t FROM Todo t WHERE t.completed = false AND t.dueDate < :now")
    List<Todo> findOverdueTodos(LocalDateTime now);
    
    // paginated query
    Page<Todo> findByUserId(Long userId, Pageable pageable);
}
```

### Service Layer

```java
package com.example.todoapp.service;

import com.example.todoapp.dto.TodoDTO;
import com.example.todoapp.dto.CreateTodoRequest;
import com.example.todoapp.dto.UpdateTodoRequest;
import com.example.todoapp.entity.Todo;
import com.example.todoapp.entity.User;
import com.example.todoapp.exception.ResourceNotFoundException;
import com.example.todoapp.mapper.TodoMapper;
import com.example.todoapp.repository.TodoRepository;
import com.example.todoapp.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Transactional
public class TodoService {
    
    private final TodoRepository todoRepository;
    private final UserRepository userRepository;
    private final TodoMapper todoMapper;
    
    @Transactional(readOnly = true)
    public List<TodoDTO> getAllTodos() {
        return todoRepository.findAll()
                .stream()
                .map(todoMapper::toDTO)
                .collect(Collectors.toList());
    }
    
    @Transactional(readOnly = true)
    public TodoDTO getTodoById(Long id) {
        Todo todo = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found with id: " + id));
        return todoMapper.toDTO(todo);
    }
    
    public TodoDTO createTodo(CreateTodoRequest request) {
        // get current user (from security context in real app)
        User user = userRepository.findById(1L)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        
        Todo todo = todoMapper.toEntity(request);
        todo.setUser(user);
        
        Todo savedTodo = todoRepository.save(todo);
        return todoMapper.toDTO(savedTodo);
    }
    
    public TodoDTO updateTodo(Long id, UpdateTodoRequest request) {
        Todo todo = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found with id: " + id));
        
        todoMapper.updateEntityFromDTO(todo, request);
        
        Todo updatedTodo = todoRepository.save(todo);
        return todoMapper.toDTO(updatedTodo);
    }
    
    public void deleteTodo(Long id) {
        if (!todoRepository.existsById(id)) {
            throw new ResourceNotFoundException("Todo not found with id: " + id);
        }
        todoRepository.deleteById(id);
    }
}
```

### Controller Layer (Complete)

```java
package com.example.todoapp.controller;

import com.example.todoapp.dto.TodoDTO;
import com.example.todoapp.dto.CreateTodoRequest;
import com.example.todoapp.dto.UpdateTodoRequest;
import com.example.todoapp.service.TodoService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    @GetMapping
    public ResponseEntity<List<TodoDTO>> getAllTodos() {
        List<TodoDTO> todos = todoService.getAllTodos();
        return ResponseEntity.ok(todos);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<TodoDTO> getTodoById(@PathVariable Long id) {
        TodoDTO todo = todoService.getTodoById(id);
        return ResponseEntity.ok(todo);
    }
    
    @PostMapping
    public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
        TodoDTO todo = todoService.createTodo(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(todo);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<TodoDTO> updateTodo(
            @PathVariable Long id,
            @Valid @RequestBody UpdateTodoRequest request) {
        TodoDTO todo = todoService.updateTodo(id, request);
        return ResponseEntity.ok(todo);
    }
    
    @DeleteMapping("/{id}")
```

## Part 3: Advanced Controller Patterns

### Pagination and Sorting

```java
@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    // Paginated endpoint
    @GetMapping("/paginated")
    public ResponseEntity<Page<TodoDTO>> getTodosPaginated(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "ASC") String direction) {
        
        Page<TodoDTO> todos = todoService.getTodosPaginated(page, size, sortBy, direction);
        return ResponseEntity.ok(todos);
    }
    
    // Using Pageable parameter (Spring auto-maps from query params)
    @GetMapping("/page")
    public ResponseEntity<Page<TodoDTO>> getTodosPage(Pageable pageable) {
        Page<TodoDTO> todos = todoService.getTodosPage(pageable);
        return ResponseEntity.ok(todos);
    }
}
```

### Filtering and Search

```java
@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    // query parameters for filtering
    @GetMapping("/search")
    public ResponseEntity<List<TodoDTO>> searchTodos(
            @RequestParam(required = false) String title,
            @RequestParam(required = false) Boolean completed,
            @RequestParam(required = false) String priority) {
        
        List<TodoDTO> todos = todoService.searchTodos(title, completed, priority);
        return ResponseEntity.ok(todos);
    }
    
    // get by status
    @GetMapping("/status/{completed}")
    public ResponseEntity<List<TodoDTO>> getTodosByStatus(
            @PathVariable Boolean completed) {
        List<TodoDTO> todos = todoService.getTodosByCompleted(completed);
        return ResponseEntity.ok(todos);
    }
    
    // get by priority
    @GetMapping("/priority/{priority}")
    public ResponseEntity<List<TodoDTO>> getTodosByPriority(
            @PathVariable String priority) {
        List<TodoDTO> todos = todoService.getTodosByPriority(priority);
        return ResponseEntity.ok(todos);
    }
}
```

### Bulk Operations

```java
@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    // bulk create
    @PostMapping("/bulk")
    public ResponseEntity<List<TodoDTO>> createTodosBulk(
            @Valid @RequestBody List<CreateTodoRequest> requests) {
        List<TodoDTO> todos = todoService.createTodosBulk(requests);
        return ResponseEntity.status(HttpStatus.CREATED).body(todos);
    }
    
    // bulk delete
    @DeleteMapping("/bulk")
    public ResponseEntity<Void> deleteTodosBulk(@RequestBody List<Long> ids) {
        todoService.deleteTodosBulk(ids);
        return ResponseEntity.noContent().build();
    }
    
    // mark all as complete
    @PatchMapping("/complete-all")
    public ResponseEntity<Void> markAllComplete() {
        todoService.markAllComplete();
        return ResponseEntity.ok().build();
    }
}
```

### Custom Response Wrappers

```java
// generic API Response wrapper
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;
    
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, LocalDateTime.now());
    }
    
    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>(true, message, data, LocalDateTime.now());
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null, LocalDateTime.now());
    }
}

// use in controller
@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    @PostMapping
    public ResponseEntity<ApiResponse<TodoDTO>> createTodo(
            @Valid @RequestBody CreateTodoRequest request) {
        TodoDTO todo = todoService.createTodo(request);
        ApiResponse<TodoDTO> response = ApiResponse.success("Todo created successfully", todo);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```
## Part 4: Authorization Annotations
```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    
    // only authenticated users can access
    @PreAuthorize("isAuthenticated()")
    @GetMapping
    public ResponseEntity<List<TodoDTO>> getAllTodos() {
        // if user is not logged in, spring security blocks this
        // and returns 401/403 before method executes
    }
    
    // only users with ADMIN role can access
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id) {
        todoService.deleteTodo(id);
        return ResponseEntity.noContent().build();
    }
    
    // only the owner can update their own todo
    @PreAuthorize("@todoService.isOwner(#id, authentication.name)")
    @PutMapping("/{id}")
    public ResponseEntity<TodoDTO> updateTodo(
            @PathVariable Long id,
            @RequestBody UpdateTodoRequest request) {
        TodoDTO todo = todoService.updateTodo(id, request);
        return ResponseEntity.ok(todo);
    }
}

```
## Part 5: Common Pitfalls

### Business Logic in Controllers

```java
// DON'T: Business logic in controller
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    // Don't do validation logic here
    if (request.getTitle() == null || request.getTitle().isEmpty()) {
        throw new BadRequestException("Title is required");
    }
    
    // don't do business logic here
    Todo todo = new Todo();
    todo.setTitle(request.getTitle());
    todo.setCompleted(false);
    
    // dn't call repository directly
    Todo saved = todoRepository.save(todo);
    
    return ResponseEntity.ok(todoMapper.toDTO(saved));
}

// INSTEAD: delegate to service layer
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}
```

### Returning Entities Instead of DTOs

```java
// BAD: exposing entity directly
@GetMapping("/{id}")
public ResponseEntity<Todo> getTodoById(@PathVariable Long id) {
    Todo todo = todoService.getTodoById(id); // Returns entity
    return ResponseEntity.ok(todo);
    // problems:
    // 1. exposes internal structure
    // 2. may cause lazy loading exceptions
    // 3. circular reference issues with Jackson
    // 4. can't control what data is sent to client
}

// GOOD: Return DTO
@GetMapping("/{id}")
public ResponseEntity<TodoDTO> getTodoById(@PathVariable Long id) {
    TodoDTO todo = todoService.getTodoById(id); // Returns DTO
    return ResponseEntity.ok(todo);
}
```

### Not Using Proper HTTP Status Codes

```java
// BAD: always returning 200 OK
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.ok(todo); // should be 201 Created
}

@DeleteMapping("/{id}")
public ResponseEntity<String> deleteTodo(@PathVariable Long id) {
    todoService.deleteTodo(id);
    return ResponseEntity.ok("Deleted"); // should be 204 No Content
}

// INSTEAD: use appropriate status codes
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo); // 201
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteTodo(@PathVariable Long id) {
    todoService.deleteTodo(id);
    return ResponseEntity.noContent().build(); // 204
}
```

### Missing @Valid Annotation

```java
// BAD: validation not triggered
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    // Validation annotations in DTO are ignored!
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}

// INSTEAD: vnable validation with @Valid
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}
```

### Not Handling Transaction Boundaries Properly

```java
// BAD: transaction in controller
@PostMapping
@Transactional // Don't use transactions in controllers
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}

// INSTEAD: transactions in service layer
// controller
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) {
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}

// service
@Service
@Transactional // transaction management in service layer
public class TodoService {
    public TodoDTO createTodo(CreateTodoRequest request) {
        // business logic with transaction
    }
}
```

### Exposing Too Much in Error Messages

```java
// BAD: exposing stack traces and sensitive info
@ExceptionHandler(Exception.class)
public ResponseEntity<String> handleException(Exception ex) {
    return ResponseEntity
        .status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(ex.getMessage() + "\n" + Arrays.toString(ex.getStackTrace()));
    // exposes internal implementation details!
}

// INSTEAD: generic error messages for clients
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleException(Exception ex) {
    // log the full error server-side
    logger.error("Unexpected error", ex);
    
    // return generic message to client
    ErrorResponse error = new ErrorResponse(
        HttpStatus.INTERNAL_SERVER_ERROR.value(),
        "An unexpected error occurred. Please try again later.",
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
}
```

### Not Using @PathVariable and @RequestParam Correctly

```java
// BAD: wrong annotation usage
@GetMapping("/{id}")
public ResponseEntity<TodoDTO> getTodoById(@RequestParam Long id) {
    // @RequestParam expects query parameter: /api/todos?id=1
    // But URL pattern expects path variable: /api/todos/1
}

// INSTEAD: use correct annotations
@GetMapping("/{id}")
public ResponseEntity<TodoDTO> getTodoById(@PathVariable Long id) {
    // Correctly extracts from path: /api/todos/1
    TodoDTO todo = todoService.getTodoById(id);
    return ResponseEntity.ok(todo);
}

@GetMapping("/search")
public ResponseEntity<List<TodoDTO>> searchTodos(@RequestParam String keyword) {
    // correctly extracts from query: /api/todos/search?keyword=groceries
    List<TodoDTO> todos = todoService.search(keyword);
    return ResponseEntity.ok(todos);
}
```

## Part 6: Best Practices

### Use Constructor Injection (Lombok)

```java
// GOOD: constructor injection with Lombok
@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor // generates constructor for final fields
public class TodoController {
    
    private final TodoService todoService;
    // immutable, can't be changed after construction
}

// AVOID: field injection
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    
    @Autowired // hard to test, mutable
    private TodoService todoService;
}
```

### Keep Controllers Thin

```java
// controller should only:
// 1. receive request
// 2. validate input (via @Valid)
// 3. call service
// 4. return response

@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {
    
    private final TodoService todoService;
    
    @PostMapping
    public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
        TodoDTO todo = todoService.createTodo(request); // Delegate to service
        return ResponseEntity.status(HttpStatus.CREATED).body(todo);
    }
}
```

### Use Proper REST Conventions

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    
    // GET /api/todos - list all
    @GetMapping
    public ResponseEntity<List<TodoDTO>> getAllTodos() { }
    
    // GET /api/todos/1 - get one
    @GetMapping("/{id}")
    public ResponseEntity<TodoDTO> getTodoById(@PathVariable Long id) { }
    
    // POST /api/todos - create
    @PostMapping
    public ResponseEntity<TodoDTO> createTodo(@RequestBody CreateTodoRequest request) { }
    
    // PUT /api/todos/1 - full update
    @PutMapping("/{id}")
    public ResponseEntity<TodoDTO> updateTodo(@PathVariable Long id, @RequestBody UpdateTodoRequest request) { }
    
    // PATCH /api/todos/1 - partial update
    @PatchMapping("/{id}")
    public ResponseEntity<TodoDTO> partialUpdateTodo(@PathVariable Long id, @RequestBody Map<String, Object> updates) { }
    
    // DELETE /api/todos/1 - delete
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id) { }
}
```

### Use API Versioning

```java
// option 1: URL versioning (most common)
@RestController
@RequestMapping("/api/v1/todos")
public class TodoControllerV1 { }

@RestController
@RequestMapping("/api/v2/todos")
public class TodoControllerV2 { }

// Option 2: header versioning
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    
    @GetMapping(headers = "API-Version=1")
    public ResponseEntity<List<TodoDTO>> getAllTodosV1() { }
    
    @GetMapping(headers = "API-Version=2")
    public ResponseEntity<List<TodoDTOV2>> getAllTodosV2() { }
}
```

### HTTP Status Codes

| Status | Use Case | Example |
|--------|----------|---------|
| 200 OK | Successful GET, PUT, PATCH | Get todo, Update todo |
| 201 Created | Successful POST | Create todo |
| 204 No Content | Successful DELETE | Delete todo |
| 400 Bad Request | Validation error | Invalid input |
| 401 Unauthorized | Not authenticated | Missing/invalid token |
| 403 Forbidden | Not authorized | No permission |
| 404 Not Found | Resource doesn't exist | Todo not found |
| 500 Internal Server Error | Server error | Unexpected exception |

### Key Annotations Reference

```java
// Class-level
@RestController              // marks as REST controller
@RequestMapping("/api/path") // base URL path
@RequiredArgsConstructor    // constructor injection (Lombok)

// Method-level
@GetMapping                 // HTTP GET
@PostMapping                // HTTP POST
@PutMapping                 // HTTP PUT
@PatchMapping              // HTTP PATCH
@DeleteMapping             // HTTP DELETE

// Parameter-level
@PathVariable              // from URL path
@RequestParam              // from query string
@RequestBody               // from request body (JSON)
@Valid                     // enable validation

// Exception handling
@ExceptionHandler          // handle specific exceptions
@RestControllerAdvice      // global exception handler

// strings
@NotNull, @NotEmpty, @NotBlank
@Size(min = x, max = y)
@Pattern(regexp = "...")
@Email

// numbers
@Min(x), @Max(x)
@Positive, @PositiveOrZero
@Negative, @NegativeOrZero
@DecimalMin("x"), @DecimalMax("x")

// dates
@Past, @PastOrPresent
@Future, @FutureOrPresent

// collections
@NotEmpty
@Size(min = x, max = y)

// boolean
@AssertTrue, @AssertFalse

// nested
@Valid
```



### How @Valid Works

```java
// 1. define validation rules in your DTO
public class CreateTodoRequest {
    
    @NotBlank(message = "Title is required")
    @Size(min = 1, max = 100, message = "Title must be between 1 and 100 characters")
    private String title;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @NotNull(message = "Priority is required")
    @Pattern(regexp = "LOW|MEDIUM|HIGH", message = "Priority must be LOW, MEDIUM, or HIGH")
    private String priority;
    
    @Email(message = "Invalid email format")
    private String assignedTo;
    
    @Future(message = "Due date must be in the future")
    private LocalDateTime dueDate;
    
    @Min(value = 0, message = "Estimated hours must be positive")
    @Max(value = 1000, message = "Estimated hours cannot exceed 1000")
    private Integer estimatedHours;
}

// 2. use @Valid in controller
@PostMapping
public ResponseEntity<TodoDTO> createTodo(@Valid @RequestBody CreateTodoRequest request) {
    // Validation happens automatically before this executes
    TodoDTO todo = todoService.createTodo(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(todo);
}

// 3. handle validation errors globally
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity.badRequest().body(errors);
    }
}
```

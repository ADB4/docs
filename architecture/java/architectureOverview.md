# Layered Backend Architecture in Java/Spring/Postgres

## Table of Contents
- [Introduction](#introduction)
- [Architecture Overview](#architecture-overview)
- [Layer Responsibilities](#layer-responsibilities)
- [Entity Layer](#entity-layer)
- [Repository Layer](#repository-layer)
- [DTO Layer](#dto-layer)
- [Mapper Layer](#mapper-layer)
- [Service Layer](#service-layer)
- [Controller Layer](#controller-layer)
- [Data Flow](#data-flow)
- [Common Patterns](#common-patterns)
- [Anti-Patterns](#anti-patterns)
- [Testing Strategy](#testing-strategy)
- [Performance Considerations](#performance-considerations)

---

## Introduction

Layered architecture organizes backend applications into distinct horizontal layers, each with specific responsibilities. This pattern promotes separation of concerns, making applications more maintainable, testable, and scalable.

A standard Spring Boot application employs six primary layers:
1. Controller Layer (Presentation)
2. DTO Layer (Data Transfer)
3. Service Layer (Business Logic)
4. Mapper Layer (Object Transformation)
5. Repository Layer (Data Access)
6. Entity Layer (Domain Model)

Each layer communicates only with adjacent layers, preventing tight coupling and maintaining clear boundaries.

### Java Version Note

This guide uses Java 21 features, specifically Java records for DTOs. Records provide immutable data carriers with concise syntax and are ideal for data transfer objects. Entities continue to use JPA annotations with traditional classes, as JPA entities require mutable state for the persistence framework.

---

## Architecture Overview

```
Client Request
      ↓
┌─────────────────┐
│   Controller    │  ← HTTP endpoints, request validation, response formatting
└─────────────────┘
      ↓
┌─────────────────┐
│      DTO        │  ← Data transfer objects for API contracts
└─────────────────┘
      ↓
┌─────────────────┐
│    Service      │  ← Business logic, transaction management, orchestration
└─────────────────┘
      ↓
┌─────────────────┐
│     Mapper      │  ← Entity ↔ DTO transformation
└─────────────────┘
      ↓
┌─────────────────┐
│   Repository    │  ← Data access, query methods
└─────────────────┘
      ↓
┌─────────────────┐
│     Entity      │  ← Domain model, database mapping
└─────────────────┘
      ↓
   Database
```

### Communication Rules

- Controllers receive DTOs and return DTOs
- Services work with both DTOs (from controllers) and Entities (from repositories)
- Mappers translate between DTOs and Entities
- Repositories work exclusively with Entities
- Entities represent database tables

---

## Layer Responsibilities

### Quick Reference Table

| Layer | Responsible For | NOT Responsible For |
|-------|----------------|---------------------|
| Controller | HTTP handling, request validation, response formatting | Business logic, database queries, object mapping |
| DTO | Data structure definition, validation annotations | Business logic, persistence, transformation logic |
| Service | Business logic, transaction coordination, orchestration | HTTP concerns, SQL queries, DTO structure |
| Mapper | Entity ↔ DTO transformation | Business logic, validation, persistence |
| Repository | Database queries, CRUD operations | Business logic, DTO conversion, transaction management |
| Entity | Domain model, database mapping | API contracts, business rules, HTTP concerns |

---

## Entity Layer

### Purpose

Entities represent domain objects and map directly to database tables. They define the structure of persisted data and relationships between domain objects.

### Responsibilities

- Define database table structure
- Specify column mappings and constraints
- Declare entity relationships (OneToMany, ManyToOne, etc.)
- Provide getter/setter methods for persistence framework
- Include JPA annotations for ORM mapping

### NOT Responsible For

- Business logic or validation rules
- API response formatting
- Data transformation for external systems
- Exposing sensitive data to API consumers
- Containing service or repository logic

### Example

```java
package com.example.todos.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;

@Entity
@Table(name = "todos")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TodoEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 500)
    private String text;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(nullable = false)
    private Boolean completed = false;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TodoPriority priority;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private UserEntity user;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private CategoryEntity category;
    
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
```

```java
package com.example.todos.entity;

public enum TodoPriority {
    LOW,
    MEDIUM,
    HIGH
}
```

### Key Considerations

**Lazy Loading:**
`FetchType.LAZY` should be used for relationships to avoid N+1 query problems. Eager loading should be explicit in repository queries when needed.

**Immutable Fields:**
`updatable = false` should be used for fields that should not change after creation (e.g., createdAt, id).

**Naming Conventions:**
- Entity class names typically end with "Entity" to distinguish from DTOs
- Table and column names use snake_case
- Java field names use camelCase

**Avoid:**
- Business logic methods in entities
- Validation annotations (use DTOs for validation)
- Exposing entities directly in API responses
- Circular references that cause infinite recursion

---

## Repository Layer

### Purpose

The Repository layer provides an abstraction over database operations. It encapsulates data access logic and provides a collection-like interface for domain objects.

### Responsibilities

- Define database query methods
- Implement custom queries using JPQL or native SQL
- Provide CRUD operations
- Handle database-specific concerns
- Execute queries and return entities

### NOT Responsible For

- Business logic or validation
- Transaction management (handled by Service layer)
- DTO conversion
- Error handling beyond database exceptions
- Complex data manipulation

### Example

```java
package com.example.todos.repository;

import com.example.todos.entity.TodoEntity;
import com.example.todos.entity.TodoPriority;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@Repository
public interface TodoRepository extends JpaRepository<TodoEntity, Long> {
    
    // Spring Data JPA derived queries
    List<TodoEntity> findByUserId(Long userId);
    
    List<TodoEntity> findByUserIdAndCompleted(Long userId, Boolean completed);
    
    List<TodoEntity> findByUserIdAndPriority(Long userId, TodoPriority priority);
    
    Optional<TodoEntity> findByIdAndUserId(Long id, Long userId);
    
    Long countByUserIdAndCompleted(Long userId, Boolean completed);
    
    // Custom JPQL queries
    @Query("SELECT t FROM TodoEntity t WHERE t.user.id = :userId AND t.completed = false " +
           "ORDER BY CASE t.priority WHEN 'HIGH' THEN 1 WHEN 'MEDIUM' THEN 2 " +
           "WHEN 'LOW' THEN 3 END, t.createdAt ASC")
    List<TodoEntity> findActiveByPriority(@Param("userId") Long userId);
    
    @Query("SELECT t FROM TodoEntity t WHERE t.user.id = :userId " +
           "AND t.completed = false AND t.createdAt < :date")
    List<TodoEntity> findOverdue(@Param("userId") Long userId, @Param("date") LocalDateTime date);
    
    // Query with JOIN FETCH to avoid N+1 problem
    @Query("SELECT t FROM TodoEntity t " +
           "LEFT JOIN FETCH t.category " +
           "WHERE t.user.id = :userId")
    List<TodoEntity> findByUserIdWithCategory(@Param("userId") Long userId);
    
    // Native SQL query for complex operations
    @Query(value = "SELECT * FROM todos t WHERE t.user_id = :userId " +
                   "AND LOWER(t.text) LIKE LOWER(CONCAT('%', :searchTerm, '%'))",
           nativeQuery = true)
    List<TodoEntity> searchByText(@Param("userId") Long userId, 
                                  @Param("searchTerm") String searchTerm);
    
    // Bulk operations
    void deleteByUserIdAndCompleted(Long userId, Boolean completed);
}
```

### Key Considerations

**Query Method Naming:**
Spring Data JPA derives queries from method names. The naming convention follows: `findBy`, `countBy`, `deleteBy` followed by field names and conditions.

**Custom Queries:**
`@Query` should be used for complex queries that cannot be expressed through method names. JPQL is preferred over native SQL for database portability.

**JOIN FETCH:**
We use `JOIN FETCH` in JPQL queries to eagerly load relationships and avoid N+1 query problems.

**Avoid:**
- Business logic in repository methods
- Transaction management annotations (use Service layer)
- Returning DTOs from repository methods
- Complex data transformation
- Caching logic (use Service layer)

---

## DTO Layer

### Purpose

Data Transfer Objects (DTOs) define the API contract between client and server. They control what data is exposed externally and provide a stable interface independent of internal entity structure.

### Responsibilities

- Define API request/response structure
- Include validation annotations
- Provide documentation through annotations
- Serve as version-stable contracts
- Exclude sensitive or internal fields
- Maintain immutability (using records)

### NOT Responsible For

- Database mapping
- Business logic
- Object transformation
- Persistence operations
- Maintaining entity relationships

### Example

```java
package com.example.todos.dto;

import com.example.todos.entity.TodoPriority;
import com.fasterxml.jackson.annotation.JsonFormat;
import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.*;

import java.time.LocalDateTime;

// Request DTO - for creating todos
@Schema(description = "Request object for creating a new todo")
public record CreateTodoRequest(
    
    @NotBlank(message = "Text is required")
    @Size(min = 1, max = 500, message = "Text must be between 1 and 500 characters")
    @Schema(description = "Todo text", example = "Complete project documentation")
    String text,
    
    @Size(max = 5000, message = "Description cannot exceed 5000 characters")
    @Schema(description = "Todo description", example = "Write comprehensive documentation for the API")
    String description,
    
    @NotNull(message = "Priority is required")
    @Schema(description = "Todo priority level", example = "HIGH")
    TodoPriority priority,
    
    @Schema(description = "Category ID for the todo", example = "1")
    Long categoryId
) {}

// Request DTO - for updating todos
@Schema(description = "Request object for updating an existing todo")
public record UpdateTodoRequest(
    
    @Size(min = 1, max = 500, message = "Text must be between 1 and 500 characters")
    @Schema(description = "Todo text", example = "Complete project documentation")
    String text,
    
    @Size(max = 5000, message = "Description cannot exceed 5000 characters")
    @Schema(description = "Todo description")
    String description,
    
    @Schema(description = "Todo completion status")
    Boolean completed,
    
    @Schema(description = "Todo priority level")
    TodoPriority priority,
    
    @Schema(description = "Category ID for the todo")
    Long categoryId
) {}

// Response DTO - for returning todo data
@Schema(description = "Todo response object")
public record TodoResponse(
    
    @Schema(description = "Todo ID", example = "1")
    Long id,
    
    @Schema(description = "Todo text", example = "Complete project documentation")
    String text,
    
    @Schema(description = "Todo description")
    String description,
    
    @Schema(description = "Todo completion status", example = "false")
    Boolean completed,
    
    @Schema(description = "Todo priority level", example = "HIGH")
    TodoPriority priority,
    
    @Schema(description = "Todo creation timestamp")
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime createdAt,
    
    @Schema(description = "Todo last update timestamp")
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime updatedAt,
    
    @Schema(description = "Category information")
    CategorySummary category
) {
    public record CategorySummary(Long id, String name) {}
}

// Summary DTO - for list responses
@Schema(description = "Todo summary for list views")
public record TodoSummary(
    
    @Schema(description = "Todo ID")
    Long id,
    
    @Schema(description = "Todo text")
    String text,
    
    @Schema(description = "Todo completion status")
    Boolean completed,
    
    @Schema(description = "Todo priority level")
    TodoPriority priority,
    
    @Schema(description = "Category name")
    String categoryName
) {}
```

### Key Considerations

**Using Records for DTOs:**
Java records (introduced in Java 14, standard in Java 16+) are ideal for DTOs. They provide immutability, concise syntax, and automatic implementation of equals(), hashCode(), and toString(). Records work seamlessly with validation annotations and JSON serialization.

**Separation of Request and Response:**
We use separate records for requests and responses. Request DTOs contain validation; response DTOs control what data is exposed.

**Validation Annotations:**
We apply validation at the DTO level using Jakarta Validation annotations. This ensures invalid data never reaches the service layer. Annotations work directly on record components.

**Nested Records:**
We use nested records for related entities rather than exposing full entity graphs. This prevents over-fetching and circular reference issues.

**Immutability:**
Records are immutable by default, which prevents accidental modification and makes DTOs safer for concurrent operations.

**Avoid:**
- Including entity references in DTOs
- Database-specific annotations
- Business logic methods
- Exposing internal IDs unnecessarily
- Making DTOs mutable (use records, not classes with setters)

---

## Mapper Layer

### Purpose

Mappers transform data between DTOs and Entities. They provide a clean separation between the API contract and internal domain model, allowing each to evolve independently.

### Responsibilities

- Convert Entities to DTOs
- Convert DTOs to Entities
- Handle nested object transformation
- Apply default values during conversion
- Aggregate data from multiple sources

### NOT Responsible For

- Business logic or validation
- Database operations
- Transaction management
- Complex calculations
- API request/response handling

### Example

```java
package com.example.todos.mapper;

import com.example.todos.dto.*;
import com.example.todos.entity.CategoryEntity;
import com.example.todos.entity.TodoEntity;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class TodoMapper {
    
    /**
     * Convert CreateTodoRequest to TodoEntity
     * Does not set: id, createdAt, updatedAt (handled by entity)
     * Requires: user to be set by service layer
     */
    public TodoEntity toEntity(CreateTodoRequest request) {
        TodoEntity entity = new TodoEntity();
        entity.setText(request.text());
        entity.setDescription(request.description());
        entity.setPriority(request.priority());
        entity.setCompleted(false); // Default value
        // user and category must be set by service layer
        return entity;
    }
    
    /**
     * Update existing TodoEntity with UpdateTodoRequest
     * Only updates non-null fields from request
     */
    public void updateEntity(TodoEntity entity, UpdateTodoRequest request) {
        if (request.text() != null) {
            entity.setText(request.text());
        }
        if (request.description() != null) {
            entity.setDescription(request.description());
        }
        if (request.completed() != null) {
            entity.setCompleted(request.completed());
        }
        if (request.priority() != null) {
            entity.setPriority(request.priority());
        }
        // category updated by service layer if categoryId is present
    }
    
    /**
     * Convert TodoEntity to TodoResponse
     * Includes nested category information if present
     */
    public TodoResponse toResponse(TodoEntity entity) {
        TodoResponse.CategorySummary categoryInfo = null;
        
        // Map nested category
        if (entity.getCategory() != null) {
            CategoryEntity category = entity.getCategory();
            categoryInfo = new TodoResponse.CategorySummary(
                category.getId(),
                category.getName()
            );
        }
        
        return new TodoResponse(
            entity.getId(),
            entity.getText(),
            entity.getDescription(),
            entity.getCompleted(),
            entity.getPriority(),
            entity.getCreatedAt(),
            entity.getUpdatedAt(),
            categoryInfo
        );
    }
    
    /**
     * Convert TodoEntity to TodoSummary
     * Lighter version for list responses
     */
    public TodoSummary toSummary(TodoEntity entity) {
        return new TodoSummary(
            entity.getId(),
            entity.getText(),
            entity.getCompleted(),
            entity.getPriority(),
            entity.getCategory() != null ? entity.getCategory().getName() : null
        );
    }
    
    /**
     * Convert list of TodoEntity to list of TodoResponse
     */
    public List<TodoResponse> toResponseList(List<TodoEntity> entities) {
        return entities.stream()
            .map(this::toResponse)
            .collect(Collectors.toList());
    }
    
    /**
     * Convert list of TodoEntity to list of TodoSummary
     */
    public List<TodoSummary> toSummaryList(List<TodoEntity> entities) {
        return entities.stream()
            .map(this::toSummary)
            .collect(Collectors.toList());
    }
}
```

### Alternative: Using MapStruct

```java
package com.example.todos.mapper;

import com.example.todos.dto.*;
import com.example.todos.entity.TodoEntity;
import org.mapstruct.*;

import java.util.List;

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface TodoMapperMapStruct {
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "user", ignore = true)
    @Mapping(target = "category", ignore = true)
    @Mapping(target = "completed", constant = "false")
    TodoEntity toEntity(CreateTodoRequest request);
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "user", ignore = true)
    @Mapping(target = "category", ignore = true)
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateEntity(@MappingTarget TodoEntity entity, UpdateTodoRequest request);
    
    @Mapping(target = "category", source = "category")
    TodoResponse toResponse(TodoEntity entity);
    
    @Mapping(target = "categoryName", source = "category.name")
    TodoSummary toSummary(TodoEntity entity);
    
    List<TodoResponse> toResponseList(List<TodoEntity> entities);
    
    List<TodoSummary> toSummaryList(List<TodoEntity> entities);
}
```

### Key Considerations

**Null Handling:**
Mappers must handle null values appropriately. For updates, null typically means "don't change this field."

**Record Accessors:**
Records use accessor methods without the "get" prefix: `request.text()` instead of `request.getText()`. Entity getters still use the traditional "get" prefix.

**Defensive Copying:**
When mapping collections, new lists should be created rather than exposing internal collections.

**Lazy Loading:**
Caution is required when mapping entities with lazy-loaded relationships. Access to lazy collections outside a transaction will cause LazyInitializationException.

**Avoid:**
- Business logic in mapping methods
- Database queries in mappers
- Validation logic
- Complex calculations
- Calling service methods from mappers

---

## Service Layer

### Purpose

The Service layer contains business logic and orchestrates operations across multiple repositories. It enforces business rules, manages transactions, and coordinates complex workflows.

### Responsibilities

- Implement business logic and rules
- Manage transactions
- Coordinate multiple repository operations
- Validate business constraints
- Handle complex domain operations
- Orchestrate mapper calls

### NOT Responsible For

- HTTP request/response handling
- DTO validation (handled by framework)
- SQL query construction
- DTO structure definition
- Direct database access

### Example

```java
package com.example.todos.service;

import com.example.todos.dto.*;
import com.example.todos.entity.CategoryEntity;
import com.example.todos.entity.TodoEntity;
import com.example.todos.entity.TodoPriority;
import com.example.todos.entity.UserEntity;
import com.example.todos.exception.*;
import com.example.todos.mapper.TodoMapper;
import com.example.todos.repository.CategoryRepository;
import com.example.todos.repository.TodoRepository;
import com.example.todos.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.List;

@Service
@RequiredArgsConstructor
@Slf4j
public class TodoService {
    
    private final TodoRepository todoRepository;
    private final UserRepository userRepository;
    private final CategoryRepository categoryRepository;
    private final TodoMapper todoMapper;
    
    /**
     * Retrieve all todos for a user
     * Read-only operation, no transaction needed
     */
    @Transactional(readOnly = true)
    public List<TodoSummary> getUserTodos(Long userId) {
        log.debug("Fetching todos for user: {}", userId);
        
        // Verify user exists
        verifyUserExists(userId);
        
        List<TodoEntity> todos = todoRepository.findByUserIdWithCategory(userId);
        return todoMapper.toSummaryList(todos);
    }
    
    /**
     * Retrieve active todos sorted by priority
     */
    @Transactional(readOnly = true)
    public List<TodoResponse> getActiveByPriority(Long userId) {
        log.debug("Fetching active todos by priority for user: {}", userId);
        
        verifyUserExists(userId);
        
        List<TodoEntity> todos = todoRepository.findActiveByPriority(userId);
        return todoMapper.toResponseList(todos);
    }
    
    /**
     * Get a single todo by ID
     */
    @Transactional(readOnly = true)
    public TodoResponse getTodo(Long userId, Long todoId) {
        log.debug("Fetching todo {} for user {}", todoId, userId);
        
        TodoEntity todo = findByIdAndUser(todoId, userId);
        return todoMapper.toResponse(todo);
    }
    
    /**
     * Create a new todo
     * Transactional - will rollback if any operation fails
     */
    @Transactional
    public TodoResponse createTodo(Long userId, CreateTodoRequest request) {
        log.info("Creating todo for user {}: {}", userId, request.text());
        
        // Verify user exists and get entity
        UserEntity user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + userId));
        
        // Map DTO to entity
        TodoEntity todo = todoMapper.toEntity(request);
        todo.setUser(user);
        
        // Set category if provided
        if (request.categoryId() != null) {
            CategoryEntity category = categoryRepository.findByIdAndUserId(
                request.categoryId(), userId
            ).orElseThrow(() -> new ResourceNotFoundException(
                "Category not found: " + request.categoryId()
            ));
            todo.setCategory(category);
        }
        
        // Business rule: Check user's todo limit
        long userTodoCount = todoRepository.countByUserIdAndCompleted(userId, false);
        if (userTodoCount >= 100) {
            throw new BusinessRuleException(
                "Cannot create todo: user has reached maximum of 100 active todos"
            );
        }
        
        // Save and return
        TodoEntity savedTodo = todoRepository.save(todo);
        log.info("Created todo with ID: {}", savedTodo.getId());
        
        return todoMapper.toResponse(savedTodo);
    }
    
    /**
     * Update an existing todo
     */
    @Transactional
    public TodoResponse updateTodo(Long userId, Long todoId, UpdateTodoRequest request) {
        log.info("Updating todo {} for user {}", todoId, userId);
        
        // Find existing todo
        TodoEntity todo = findByIdAndUser(todoId, userId);
        
        // Update fields
        todoMapper.updateEntity(todo, request);
        
        // Update category if provided
        if (request.categoryId() != null) {
            if (request.categoryId() == 0) {
                // Special case: 0 means remove category
                todo.setCategory(null);
            } else {
                CategoryEntity category = categoryRepository.findByIdAndUserId(
                    request.categoryId(), userId
                ).orElseThrow(() -> new ResourceNotFoundException(
                    "Category not found: " + request.categoryId()
                ));
                todo.setCategory(category);
            }
        }
        
        // Business rule: Cannot reopen high-priority completed todos after 30 days
        if (Boolean.FALSE.equals(request.completed()) && todo.getCompleted()) {
            if (todo.getPriority() == TodoPriority.HIGH) {
                LocalDateTime thirtyDaysAgo = LocalDateTime.now().minusDays(30);
                if (todo.getUpdatedAt().isBefore(thirtyDaysAgo)) {
                    throw new BusinessRuleException(
                        "Cannot reopen high-priority todo that was completed more than 30 days ago"
                    );
                }
            }
        }
        
        TodoEntity savedTodo = todoRepository.save(todo);
        return todoMapper.toResponse(savedTodo);
    }
    
    /**
     * Delete a todo
     */
    @Transactional
    public void deleteTodo(Long userId, Long todoId) {
        log.info("Deleting todo {} for user {}", todoId, userId);
        
        TodoEntity todo = findByIdAndUser(todoId, userId);
        
        // Business rule: Cannot delete high-priority active todos
        if (todo.getPriority() == TodoPriority.HIGH && !todo.getCompleted()) {
            throw new BusinessRuleException("Cannot delete active high-priority todos");
        }
        
        todoRepository.delete(todo);
        log.info("Deleted todo {}", todoId);
    }
    
    /**
     * Bulk delete all completed todos
     */
    @Transactional
    public int deleteCompleted(Long userId) {
        log.info("Deleting all completed todos for user {}", userId);
        
        verifyUserExists(userId);
        
        List<TodoEntity> completedTodos = todoRepository.findByUserIdAndCompleted(userId, true);
        int count = completedTodos.size();
        
        todoRepository.deleteByUserIdAndCompleted(userId, true);
        
        log.info("Deleted {} completed todos", count);
        return count;
    }
    
    /**
     * Get todo statistics for a user
     */
    @Transactional(readOnly = true)
    public TodoStatistics getStatistics(Long userId) {
        log.debug("Calculating todo statistics for user {}", userId);
        
        verifyUserExists(userId);
        
        List<TodoEntity> allTodos = todoRepository.findByUserId(userId);
        long totalTodos = allTodos.size();
        long completedTodos = allTodos.stream().filter(TodoEntity::getCompleted).count();
        long activeTodos = totalTodos - completedTodos;
        
        LocalDateTime weekAgo = LocalDateTime.now().minusDays(7);
        List<TodoEntity> overdueTodos = todoRepository.findOverdue(userId, weekAgo);
        
        return new TodoStatistics(totalTodos, activeTodos, completedTodos, overdueTodos.size());
    }
    
    // Private helper methods
    
    private void verifyUserExists(Long userId) {
        if (!userRepository.existsById(userId)) {
            throw new ResourceNotFoundException("User not found: " + userId);
        }
    }
    
    private TodoEntity findByIdAndUser(Long todoId, Long userId) {
        return todoRepository.findByIdAndUserId(todoId, userId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Todo not found: " + todoId + " for user: " + userId
            ));
    }
}
```

```java
package com.example.todos.dto;

public record TodoStatistics(
    Long totalTodos,
    Long activeTodos,
    Long completedTodos,
    Long overdueTodos
) {}
```

### Key Considerations

**Transaction Management:**
`@Transactional` should be used for operations that modify data. `@Transactional(readOnly = true)` should be used for read operations to optimize performance.

**Business Rules:**
All business logic belongs in the service layer. This includes validation beyond simple format checks, complex calculations, and workflow enforcement.

**Error Handling:**
Services throw domain-specific exceptions that controllers can translate into appropriate HTTP responses.

**Avoid:**
- HTTP-specific code (status codes, headers)
- Direct DTO validation
- SQL queries (use repositories)
- Direct entity exposure to controllers
- UI-specific formatting

---

## Controller Layer

### Purpose

Controllers handle HTTP requests and responses. They serve as the entry point to the application, delegating business logic to services and transforming results into HTTP responses.

### Responsibilities

- Define HTTP endpoints and routing
- Validate request format
- Invoke service layer methods
- Transform service responses to HTTP responses
- Handle HTTP-specific concerns (status codes, headers)
- Provide API documentation

### NOT Responsible For

- Business logic or validation rules
- Database operations
- Object mapping (use dedicated mappers)
- Transaction management
- Complex data transformations

### Example

```java
package com.example.todos.controller;

import com.example.todos.dto.*;
import com.example.todos.service.TodoService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users/{userId}/todos")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Todos", description = "Todo management endpoints")
public class TodoController {
    
    private final TodoService todoService;
    
    @GetMapping
    @Operation(summary = "Get all todos for a user", description = "Retrieves all todos for the specified user")
    @ApiResponse(responseCode = "200", description = "Todos retrieved successfully")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<List<TodoSummary>> getUserTodos(
            @Parameter(description = "User ID") @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/todos", userId);
        List<TodoSummary> todos = todoService.getUserTodos(userId);
        return ResponseEntity.ok(todos);
    }
    
    @GetMapping("/active")
    @Operation(summary = "Get active todos by priority", 
               description = "Retrieves active todos sorted by priority level")
    @ApiResponse(responseCode = "200", description = "Todos retrieved successfully")
    public ResponseEntity<List<TodoResponse>> getActiveByPriority(
            @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/todos/active", userId);
        List<TodoResponse> todos = todoService.getActiveByPriority(userId);
        return ResponseEntity.ok(todos);
    }
    
    @GetMapping("/{todoId}")
    @Operation(summary = "Get todo by ID", description = "Retrieves a specific todo by ID")
    @ApiResponse(responseCode = "200", description = "Todo retrieved successfully")
    @ApiResponse(responseCode = "404", description = "Todo not found")
    public ResponseEntity<TodoResponse> getTodo(
            @PathVariable Long userId,
            @Parameter(description = "Todo ID") @PathVariable Long todoId) {
        
        log.info("GET /api/users/{}/todos/{}", userId, todoId);
        TodoResponse todo = todoService.getTodo(userId, todoId);
        return ResponseEntity.ok(todo);
    }
    
    @PostMapping
    @Operation(summary = "Create a new todo", description = "Creates a new todo for the user")
    @ApiResponse(responseCode = "201", description = "Todo created successfully")
    @ApiResponse(responseCode = "400", description = "Invalid request data")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<TodoResponse> createTodo(
            @PathVariable Long userId,
            @Valid @RequestBody CreateTodoRequest request) {
        
        log.info("POST /api/users/{}/todos - {}", userId, request.text());
        TodoResponse todo = todoService.createTodo(userId, request);
        return ResponseEntity.status(HttpStatus.CREATED).body(todo);
    }
    
    @PutMapping("/{todoId}")
    @Operation(summary = "Update a todo", description = "Updates an existing todo")
    @ApiResponse(responseCode = "200", description = "Todo updated successfully")
    @ApiResponse(responseCode = "400", description = "Invalid request data")
    @ApiResponse(responseCode = "404", description = "Todo not found")
    public ResponseEntity<TodoResponse> updateTodo(
            @PathVariable Long userId,
            @PathVariable Long todoId,
            @Valid @RequestBody UpdateTodoRequest request) {
        
        log.info("PUT /api/users/{}/todos/{}", userId, todoId);
        TodoResponse todo = todoService.updateTodo(userId, todoId, request);
        return ResponseEntity.ok(todo);
    }
    
    @DeleteMapping("/{todoId}")
    @Operation(summary = "Delete a todo", description = "Deletes a specific todo")
    @ApiResponse(responseCode = "204", description = "Todo deleted successfully")
    @ApiResponse(responseCode = "404", description = "Todo not found")
    @ApiResponse(responseCode = "422", description = "Cannot delete high-priority todo")
    public ResponseEntity<Void> deleteTodo(
            @PathVariable Long userId,
            @PathVariable Long todoId) {
        
        log.info("DELETE /api/users/{}/todos/{}", userId, todoId);
        todoService.deleteTodo(userId, todoId);
        return ResponseEntity.noContent().build();
    }
    
    @DeleteMapping("/completed")
    @Operation(summary = "Delete all completed todos", 
               description = "Bulk deletes all completed todos for the user")
    @ApiResponse(responseCode = "200", description = "Completed todos deleted successfully")
    public ResponseEntity<DeleteCompletedResponse> deleteCompleted(
            @PathVariable Long userId) {
        
        log.info("DELETE /api/users/{}/todos/completed", userId);
        int deletedCount = todoService.deleteCompleted(userId);
        return ResponseEntity.ok(new DeleteCompletedResponse(deletedCount));
    }
    
    @GetMapping("/statistics")
    @Operation(summary = "Get todo statistics", 
               description = "Retrieves statistics about user's todos")
    @ApiResponse(responseCode = "200", description = "Statistics retrieved successfully")
    public ResponseEntity<TodoStatistics> getStatistics(
            @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/todos/statistics", userId);
        TodoStatistics stats = todoService.getStatistics(userId);
        return ResponseEntity.ok(stats);
    }
}
```

```java
package com.example.todos.dto;

public record DeleteCompletedResponse(int deletedCount) {}
```

### Global Exception Handler

```java
package com.example.todos.exception;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        log.error("Resource not found: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(BusinessRuleException.class)
    public ResponseEntity<ErrorResponse> handleBusinessRule(BusinessRuleException ex) {
        log.error("Business rule violation: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.UNPROCESSABLE_ENTITY.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        
        log.error("Validation error: {}", ex.getMessage());
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            LocalDateTime.now(),
            errors
        );
        
        return ResponseEntity.badRequest().body(response);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error: ", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Key Considerations

**HTTP Status Codes:**
- 200 OK: Successful GET, PUT
- 201 Created: Successful POST
- 204 No Content: Successful DELETE
- 400 Bad Request: Validation failure
- 404 Not Found: Resource doesn't exist
- 422 Unprocessable Entity: Business rule violation
- 500 Internal Server Error: Unexpected errors

**Request Validation:**
`@Valid` should be used to trigger validation on request DTOs. The framework handles validation errors.

**Path Variables:**
Path variables should be used for resource identifiers. Query parameters should be used for filters and pagination.

**Avoid:**
- Business logic in controllers
- Direct repository access
- Transaction management
- Complex data transformations
- Catching and hiding exceptions

---

## Data Flow

### Complete Request Flow Example

**Request:** `POST /api/users/1/todos`

```json
{
  "text": "Complete documentation",
  "description": "Write API documentation",
  "priority": "HIGH",
  "categoryId": 5
}
```

**Flow:**

```
1. Controller Layer
   - Receives HTTP POST request
   - Validates JSON format and structure
   - Triggers @Valid annotation validation on CreateTodoRequest
   - Validation passes
   - Calls: todoService.createTodo(userId=1, request)

2. Service Layer
   - Receives userId and CreateTodoRequest DTO
   - Begins transaction (@Transactional)
   - Verifies user exists via userRepository.findById(1)
   - Calls mapper: todoMapper.toEntity(request)
   
3. Mapper Layer
   - Receives CreateTodoRequest DTO
   - Creates new TodoEntity
   - Copies: text, description, priority using record accessors (request.text(), etc.)
   - Sets completed = false
   - Returns TodoEntity (without user/category)
   
4. Service Layer (continued)
   - Receives TodoEntity from mapper
   - Sets user on entity via userRepository result
   - Finds category via categoryRepository.findByIdAndUserId(5, 1)
   - Sets category on entity
   - Checks business rule: user has < 100 active todos
   - Business rule passes
   - Calls: todoRepository.save(entity)
   
5. Repository Layer
   - Receives TodoEntity
   - Generates SQL INSERT statement
   - Executes against database
   - Returns saved TodoEntity with generated ID
   
6. Service Layer (continued)
   - Receives saved TodoEntity from repository
   - Calls mapper: todoMapper.toResponse(entity)
   
7. Mapper Layer
   - Receives TodoEntity with full data
   - Creates TodoResponse DTO
   - Copies all fields including nested category
   - Returns TodoResponse
   
8. Service Layer (continued)
   - Returns TodoResponse to controller
   - Commits transaction
   
9. Controller Layer
   - Receives TodoResponse from service
   - Wraps in ResponseEntity with status 201 CREATED
   - Returns to client
   
10. Framework
    - Serializes TodoResponse to JSON
    - Sends HTTP response
```

**Response:** `201 Created`

```json
{
  "id": 42,
  "text": "Complete documentation",
  "description": "Write API documentation",
  "completed": false,
  "priority": "HIGH",
  "createdAt": "2024-01-15T10:30:00",
  "updatedAt": "2024-01-15T10:30:00",
  "category": {
    "id": 5,
    "name": "Work"
  }
}
```

---

## Common Patterns

### 1. Pagination

```java
// Repository
public interface TodoRepository extends JpaRepository<TodoEntity, Long> {
    Page<TodoEntity> findByUserId(Long userId, Pageable pageable);
}

// Service
@Transactional(readOnly = true)
public Page<TodoSummary> getUserTodosPaginated(Long userId, int page, int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<TodoEntity> todoPage = todoRepository.findByUserId(userId, pageable);
    return todoPage.map(todoMapper::toSummary);
}

// Controller
@GetMapping
public ResponseEntity<Page<TodoSummary>> getUserTodos(
        @PathVariable Long userId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    
    Page<TodoSummary> todos = todoService.getUserTodosPaginated(userId, page, size);
    return ResponseEntity.ok(todos);
}
```

### 2. Filtering

```java
// Repository with Specification
public interface TodoRepository extends JpaRepository<TodoEntity, Long>, 
                                       JpaSpecificationExecutor<TodoEntity> {
}

// Specification builder
public class TodoSpecifications {
    
    public static Specification<TodoEntity> hasUserId(Long userId) {
        return (root, query, cb) -> cb.equal(root.get("user").get("id"), userId);
    }
    
    public static Specification<TodoEntity> hasCompleted(Boolean completed) {
        return (root, query, cb) -> cb.equal(root.get("completed"), completed);
    }
    
    public static Specification<TodoEntity> hasPriority(TodoPriority priority) {
        return (root, query, cb) -> cb.equal(root.get("priority"), priority);
    }
}

// Service
@Transactional(readOnly = true)
public List<TodoSummary> filterTodos(Long userId, Boolean completed, TodoPriority priority) {
    Specification<TodoEntity> spec = Specification.where(TodoSpecifications.hasUserId(userId));
    
    if (completed != null) {
        spec = spec.and(TodoSpecifications.hasCompleted(completed));
    }
    if (priority != null) {
        spec = spec.and(TodoSpecifications.hasPriority(priority));
    }
    
    List<TodoEntity> todos = todoRepository.findAll(spec);
    return todoMapper.toSummaryList(todos);
}
```

### 3. Soft Delete

```java
// Entity
@Entity
@Table(name = "todos")
@SQLDelete(sql = "UPDATE todos SET deleted_at = NOW() WHERE id = ?")
@SQLRestriction("deleted_at IS NULL")
public class TodoEntity {
    // ... other fields ...
    
    @Column(name = "deleted_at")
    private LocalDateTime deletedAt;
}

// Service
@Transactional
public void deleteTodo(Long userId, Long todoId) {
    TodoEntity todo = findByIdAndUser(todoId, userId);
    todoRepository.delete(todo); // Triggers soft delete SQL
}
```

### 4. Audit Fields

```java
// Base entity for audit fields
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class AuditableEntity {
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}

// Entity extends auditable base
@Entity
@Table(name = "todos")
public class TodoEntity extends AuditableEntity {
    // ... todo-specific fields ...
}

// Enable JPA Auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            // Get current user from security context
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            return Optional.ofNullable(auth)
                .map(Authentication::getName)
                .or(() -> Optional.of("system"));
        };
    }
}
```

---

## Anti-Patterns

### 1. Anemic Domain Model

Entities should not be pure data holders. While complex business logic belongs in services, entities can contain simple domain methods.

```java
// Anti-pattern: Anemic entity
@Entity
public class TodoEntity {
    private LocalDateTime dueDate;
    private Boolean completed;
    // Only getters/setters, no behavior
}

// Service does all the work
public class TodoService {
    public boolean isOverdue(TodoEntity todo) {
        return !todo.getCompleted() && 
               todo.getDueDate().isBefore(LocalDateTime.now());
    }
}

// Better: Entity with domain methods
@Entity
public class TodoEntity {
    private LocalDateTime dueDate;
    private Boolean completed;
    
    public boolean isOverdue() {
        return !completed && dueDate != null && dueDate.isBefore(LocalDateTime.now());
    }
    
    public void markComplete() {
        this.completed = true;
    }
}
```

### 2. Service Layer Bypass

Controllers should never directly access repositories.

```java
// Anti-pattern: Controller accesses repository directly
@RestController
public class TodoController {
    private final TodoRepository todoRepository;
    
    @GetMapping("/{id}")
    public TodoEntity getTodo(@PathVariable Long id) {
        return todoRepository.findById(id).orElseThrow(); // Wrong!
    }
}

// Correct: Controller uses service
@RestController
public class TodoController {
    private final TodoService todoService;
    
    @GetMapping("/{id}")
    public TodoResponse getTodo(@PathVariable Long id) {
        return todoService.getTodo(id); // Right!
    }
}
```

### 3. Entity Exposure

Never expose entities directly in API responses.

```java
// Anti-pattern: Returning entity from controller
@GetMapping("/{id}")
public ResponseEntity<TodoEntity> getTodo(@PathVariable Long id) {
    TodoEntity todo = todoService.getTodoEntity(id); // Returns entity
    return ResponseEntity.ok(todo); // Exposes internal structure!
}

// Correct: Return DTO (record)
@GetMapping("/{id}")
public ResponseEntity<TodoResponse> getTodo(@PathVariable Long id) {
    TodoResponse todo = todoService.getTodo(id); // Returns DTO
    return ResponseEntity.ok(todo);
}
```

### 4. Transaction Mismanagement

Avoid transactions in wrong layers.

```java
// Anti-pattern: Transaction in controller
@RestController
public class TodoController {
    
    @PostMapping
    @Transactional // Wrong layer!
    public TodoResponse createTodo(@RequestBody CreateTodoRequest request) {
        // ...
    }
}

// Correct: Transaction in service
@Service
public class TodoService {
    
    @Transactional // Correct layer
    public TodoResponse createTodo(CreateTodoRequest request) {
        // ...
    }
}
```

### 5. Business Logic in Mapper

Mappers should only transform data, not make decisions.

```java
// Anti-pattern: Business logic in mapper
public class TodoMapper {
    public TodoResponse toResponse(TodoEntity entity) {
        // Mapping code...
        TodoResponse response = new TodoResponse(...);
        
        // Business logic doesn't belong here!
        if (entity.getPriority() == TodoPriority.HIGH && !entity.getCompleted()) {
            // This is wrong - mappers should only transform
        }
        return response;
    }
}

// Correct: Keep mapper simple, handle display logic in service or presentation layer
public class TodoMapper {
    public TodoResponse toResponse(TodoEntity entity) {
        TodoResponse.CategorySummary categoryInfo = null;
        if (entity.getCategory() != null) {
            categoryInfo = new TodoResponse.CategorySummary(
                entity.getCategory().getId(),
                entity.getCategory().getName()
            );
        }
        
        // Pure transformation only
        return new TodoResponse(
            entity.getId(),
            entity.getText(),
            entity.getDescription(),
            entity.getCompleted(),
            entity.getPriority(),
            entity.getCreatedAt(),
            entity.getUpdatedAt(),
            categoryInfo
        );
    }
}
```

---

## Testing Strategy

### Unit Testing by Layer

**Repository Layer Testing:**

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class TodoRepositoryTest {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void findByUserIdAndCompleted_ReturnsCorrectTodos() {
        // Given
        UserEntity user = createAndPersistUser();
        TodoEntity completedTodo = createTodo(user, true);
        TodoEntity activeTodo = createTodo(user, false);
        entityManager.persist(completedTodo);
        entityManager.persist(activeTodo);
        entityManager.flush();
        
        // When
        List<TodoEntity> completed = todoRepository.findByUserIdAndCompleted(
            user.getId(), true
        );
        
        // Then
        assertThat(completed).hasSize(1);
        assertThat(completed.get(0).getId()).isEqualTo(completedTodo.getId());
    }
}
```

**Mapper Layer Testing:**

```java
class TodoMapperTest {
    
    private TodoMapper todoMapper = new TodoMapper();
    
    @Test
    void toEntity_MapsFieldsCorrectly() {
        // Given
        CreateTodoRequest request = new CreateTodoRequest(
            "Test Todo",
            "Description",
            TodoPriority.HIGH,
            null
        );
        
        // When
        TodoEntity entity = todoMapper.toEntity(request);
        
        // Then
        assertThat(entity.getText()).isEqualTo("Test Todo");
        assertThat(entity.getDescription()).isEqualTo("Description");
        assertThat(entity.getPriority()).isEqualTo(TodoPriority.HIGH);
        assertThat(entity.getCompleted()).isFalse();
        assertThat(entity.getId()).isNull(); // Not set by mapper
    }
    
    @Test
    void toResponse_HandlesNullCategory() {
        // Given
        TodoEntity entity = createTodoEntity();
        entity.setCategory(null);
        
        // When
        TodoResponse response = todoMapper.toResponse(entity);
        
        // Then
        assertThat(response.category()).isNull();
    }
}
```

**Service Layer Testing:**

```java
@ExtendWith(MockitoExtension.class)
class TodoServiceTest {
    
    @Mock
    private TodoRepository todoRepository;
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private CategoryRepository categoryRepository;
    
    @Mock
    private TodoMapper todoMapper;
    
    @InjectMocks
    private TodoService todoService;
    
    @Test
    void createTodo_Success() {
        // Given
        Long userId = 1L;
        CreateTodoRequest request = new CreateTodoRequest(
            "Test",
            null,
            TodoPriority.MEDIUM,
            null
        );
        
        UserEntity user = new UserEntity();
        user.setId(userId);
        
        TodoEntity entity = new TodoEntity();
        TodoEntity savedEntity = new TodoEntity();
        savedEntity.setId(1L);
        
        TodoResponse expected = new TodoResponse(
            1L, "Test", null, false, TodoPriority.MEDIUM,
            LocalDateTime.now(), LocalDateTime.now(), null
        );
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(todoMapper.toEntity(request)).thenReturn(entity);
        when(todoRepository.countByUserIdAndCompleted(userId, false)).thenReturn(50L);
        when(todoRepository.save(entity)).thenReturn(savedEntity);
        when(todoMapper.toResponse(savedEntity)).thenReturn(expected);
        
        // When
        TodoResponse result = todoService.createTodo(userId, request);
        
        // Then
        assertThat(result.id()).isEqualTo(1L);
        verify(todoRepository).save(entity);
        verify(todoMapper).toResponse(savedEntity);
    }
    
    @Test
    void createTodo_ExceedsLimit_ThrowsException() {
        // Given
        Long userId = 1L;
        CreateTodoRequest request = new CreateTodoRequest(
            "Test",
            null,
            TodoPriority.MEDIUM,
            null
        );
        
        UserEntity user = new UserEntity();
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(todoRepository.countByUserIdAndCompleted(userId, false)).thenReturn(100L);
        
        // When/Then
        assertThatThrownBy(() -> todoService.createTodo(userId, request))
            .isInstanceOf(BusinessRuleException.class)
            .hasMessageContaining("maximum of 100 active todos");
        
        verify(todoRepository, never()).save(any());
    }
}
```

**Controller Layer Testing:**

```java
@WebMvcTest(TodoController.class)
class TodoControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private TodoService todoService;
    
    @Test
    void createTodo_ValidRequest_Returns201() throws Exception {
        // Given
        Long userId = 1L;
        
        TodoResponse response = new TodoResponse(
            1L,
            "Test Todo",
            null,
            false,
            TodoPriority.HIGH,
            LocalDateTime.now(),
            LocalDateTime.now(),
            null
        );
        
        when(todoService.createTodo(eq(userId), any(CreateTodoRequest.class)))
            .thenReturn(response);
        
        // When/Then
        mockMvc.perform(post("/api/users/{userId}/todos", userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "text": "Test Todo",
                        "priority": "HIGH"
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.text").value("Test Todo"));
    }
    
    @Test
    void createTodo_InvalidRequest_Returns400() throws Exception {
        // When/Then
        mockMvc.perform(post("/api/users/{userId}/todos", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "text": "",
                        "priority": "INVALID"
                    }
                    """))
            .andExpect(status().isBadRequest());
        
        verifyNoInteractions(todoService);
    }
}
```

---

## Performance Considerations

### N+1 Query Problem

```java
// Problem: N+1 queries
@Transactional(readOnly = true)
public List<TodoResponse> getTodos(Long userId) {
    List<TodoEntity> todos = todoRepository.findByUserId(userId); // 1 query
    
    return todos.stream()
        .map(todo -> {
            todo.getCategory().getName(); // N queries (lazy loading)
            return todoMapper.toResponse(todo);
        })
        .collect(Collectors.toList());
}

// Solution: Use JOIN FETCH
@Query("SELECT t FROM TodoEntity t LEFT JOIN FETCH t.category WHERE t.user.id = :userId")
List<TodoEntity> findByUserIdWithCategory(@Param("userId") Long userId);
```

### Batch Operations

```java
// Inefficient: Individual saves
public void createMultipleTodos(List<CreateTodoRequest> requests) {
    for (CreateTodoRequest request : requests) {
        TodoEntity entity = todoMapper.toEntity(request);
        todoRepository.save(entity); // Individual INSERT
    }
}

// Better: Batch save
public void createMultipleTodos(List<CreateTodoRequest> requests) {
    List<TodoEntity> entities = requests.stream()
        .map(todoMapper::toEntity)
        .collect(Collectors.toList());
    
    todoRepository.saveAll(entities); // Batch INSERT
}
```

### DTO Projection

```java
// Inefficient: Load full entities for summary
public List<TodoSummary> getTodoSummaries(Long userId) {
    List<TodoEntity> todos = todoRepository.findByUserId(userId); // Full entities
    return todoMapper.toSummaryList(todos);
}

// Better: Use projection
public interface TodoSummaryProjection {
    Long getId();
    String getText();
    Boolean getCompleted();
    TodoPriority getPriority();
}

@Query("SELECT t.id as id, t.text as text, t.completed as completed, " +
       "t.priority as priority FROM TodoEntity t WHERE t.user.id = :userId")
List<TodoSummaryProjection> findSummariesByUserId(@Param("userId") Long userId);
```

### Caching

```java
@Service
public class TodoService {
    
    // Cache user todo counts
    @Cacheable(value = "todoCounts", key = "#userId")
    @Transactional(readOnly = true)
    public Long getUserTodoCount(Long userId) {
        return todoRepository.countByUserId(userId);
    }
    
    // Evict cache on modifications
    @CacheEvict(value = "todoCounts", key = "#userId")
    @Transactional
    public TodoResponse createTodo(Long userId, CreateTodoRequest request) {
        // ...
    }
}
```

---

## Summary

Layered architecture in Spring Boot provides clear separation of concerns through distinct layers, each with specific responsibilities. The key principles are:

1. **Controllers** handle HTTP concerns only
2. **DTOs** define API contracts and validation
3. **Services** contain business logic and orchestration
4. **Mappers** transform between DTOs and Entities
5. **Repositories** manage data access
6. **Entities** represent the domain model

Maintaining strict boundaries between layers improves testability, maintainability, and allows teams to work on different layers independently. Understanding what each layer is NOT responsible for is as important as understanding its responsibilities.

Success with layered architecture requires discipline in respecting layer boundaries and avoiding the temptation to bypass layers for convenience. The initial overhead of proper layering pays dividends in long-term code quality and system evolution.
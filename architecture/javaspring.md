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

The standard Spring Boot application employs six primary layers:
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
package com.example.tasks.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;

@Entity
@Table(name = "tasks")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TaskEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 500)
    private String title;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(nullable = false)
    private Boolean completed = false;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TaskPriority priority;
    
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
package com.example.tasks.entity;

public enum TaskPriority {
    LOW,
    MEDIUM,
    HIGH,
    URGENT
}
```

### Key Considerations

**Lazy Loading:**
Use `FetchType.LAZY` for relationships to avoid N+1 query problems. Eager loading should be explicit in repository queries when needed.

**Immutable Fields:**
Use `updatable = false` for fields that should not change after creation (e.g., createdAt, id).

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
package com.example.tasks.repository;

import com.example.tasks.entity.TaskEntity;
import com.example.tasks.entity.TaskPriority;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@Repository
public interface TaskRepository extends JpaRepository<TaskEntity, Long> {
    
    // Spring Data JPA derived queries
    List<TaskEntity> findByUserId(Long userId);
    
    List<TaskEntity> findByUserIdAndCompleted(Long userId, Boolean completed);
    
    List<TaskEntity> findByUserIdAndPriority(Long userId, TaskPriority priority);
    
    Optional<TaskEntity> findByIdAndUserId(Long id, Long userId);
    
    Long countByUserIdAndCompleted(Long userId, Boolean completed);
    
    // Custom JPQL queries
    @Query("SELECT t FROM TaskEntity t WHERE t.user.id = :userId AND t.completed = false " +
           "ORDER BY CASE t.priority WHEN 'URGENT' THEN 1 WHEN 'HIGH' THEN 2 " +
           "WHEN 'MEDIUM' THEN 3 WHEN 'LOW' THEN 4 END, t.createdAt ASC")
    List<TaskEntity> findActiveTasksByPriority(@Param("userId") Long userId);
    
    @Query("SELECT t FROM TaskEntity t WHERE t.user.id = :userId " +
           "AND t.completed = false AND t.createdAt < :date")
    List<TaskEntity> findOverdueTasks(@Param("userId") Long userId, @Param("date") LocalDateTime date);
    
    // Query with JOIN FETCH to avoid N+1 problem
    @Query("SELECT t FROM TaskEntity t " +
           "LEFT JOIN FETCH t.category " +
           "WHERE t.user.id = :userId")
    List<TaskEntity> findByUserIdWithCategory(@Param("userId") Long userId);
    
    // Native SQL query for complex operations
    @Query(value = "SELECT * FROM tasks t WHERE t.user_id = :userId " +
                   "AND LOWER(t.title) LIKE LOWER(CONCAT('%', :searchTerm, '%'))",
           nativeQuery = true)
    List<TaskEntity> searchTasksByTitle(@Param("userId") Long userId, 
                                        @Param("searchTerm") String searchTerm);
    
    // Bulk operations
    void deleteByUserIdAndCompleted(Long userId, Boolean completed);
}
```

### Key Considerations

**Query Method Naming:**
Spring Data JPA derives queries from method names. Follow the naming convention: `findBy`, `countBy`, `deleteBy` followed by field names and conditions.

**Custom Queries:**
Use `@Query` for complex queries that cannot be expressed through method names. Prefer JPQL over native SQL for database portability.

**JOIN FETCH:**
Use `JOIN FETCH` in JPQL queries to eagerly load relationships and avoid N+1 query problems.

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
package com.example.tasks.dto;

import com.example.tasks.entity.TaskPriority;
import com.fasterxml.jackson.annotation.JsonFormat;
import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.*;

import java.time.LocalDateTime;

// Request DTO - for creating tasks
@Schema(description = "Request object for creating a new task")
public record CreateTaskRequest(
    
    @NotBlank(message = "Title is required")
    @Size(min = 1, max = 500, message = "Title must be between 1 and 500 characters")
    @Schema(description = "Task title", example = "Complete project documentation")
    String title,
    
    @Size(max = 5000, message = "Description cannot exceed 5000 characters")
    @Schema(description = "Task description", example = "Write comprehensive documentation for the API")
    String description,
    
    @NotNull(message = "Priority is required")
    @Schema(description = "Task priority level", example = "HIGH")
    TaskPriority priority,
    
    @Schema(description = "Category ID for the task", example = "1")
    Long categoryId
) {}

// Request DTO - for updating tasks
@Schema(description = "Request object for updating an existing task")
public record UpdateTaskRequest(
    
    @Size(min = 1, max = 500, message = "Title must be between 1 and 500 characters")
    @Schema(description = "Task title", example = "Complete project documentation")
    String title,
    
    @Size(max = 5000, message = "Description cannot exceed 5000 characters")
    @Schema(description = "Task description")
    String description,
    
    @Schema(description = "Task completion status")
    Boolean completed,
    
    @Schema(description = "Task priority level")
    TaskPriority priority,
    
    @Schema(description = "Category ID for the task")
    Long categoryId
) {}

// Response DTO - for returning task data
@Schema(description = "Task response object")
public record TaskResponse(
    
    @Schema(description = "Task ID", example = "1")
    Long id,
    
    @Schema(description = "Task title", example = "Complete project documentation")
    String title,
    
    @Schema(description = "Task description")
    String description,
    
    @Schema(description = "Task completion status", example = "false")
    Boolean completed,
    
    @Schema(description = "Task priority level", example = "HIGH")
    TaskPriority priority,
    
    @Schema(description = "Task creation timestamp")
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime createdAt,
    
    @Schema(description = "Task last update timestamp")
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime updatedAt,
    
    @Schema(description = "Category information")
    CategorySummary category
) {
    public record CategorySummary(Long id, String name) {}
}

// Summary DTO - for list responses
@Schema(description = "Task summary for list views")
public record TaskSummary(
    
    @Schema(description = "Task ID")
    Long id,
    
    @Schema(description = "Task title")
    String title,
    
    @Schema(description = "Task completion status")
    Boolean completed,
    
    @Schema(description = "Task priority level")
    TaskPriority priority,
    
    @Schema(description = "Category name")
    String categoryName
) {}
```

### Key Considerations

**Using Records for DTOs:**
Java records (introduced in Java 14, standard in Java 16+) are ideal for DTOs. They provide immutability, concise syntax, and automatic implementation of equals(), hashCode(), and toString(). Records work seamlessly with validation annotations and JSON serialization.

**Separation of Request and Response:**
Use separate records for requests and responses. Request DTOs contain validation; response DTOs control what data is exposed.

**Validation Annotations:**
Apply validation at the DTO level using Jakarta Validation annotations. This ensures invalid data never reaches the service layer. Annotations work directly on record components.

**Nested Records:**
Use nested records for related entities rather than exposing full entity graphs. This prevents over-fetching and circular reference issues.

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
package com.example.tasks.mapper;

import com.example.tasks.dto.*;
import com.example.tasks.entity.CategoryEntity;
import com.example.tasks.entity.TaskEntity;
import com.example.tasks.entity.UserEntity;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class TaskMapper {
    
    /**
     * Convert CreateTaskRequest to TaskEntity
     * Does not set: id, createdAt, updatedAt (handled by entity)
     * Requires: user to be set by service layer
     */
    public TaskEntity toEntity(CreateTaskRequest request) {
        TaskEntity entity = new TaskEntity();
        entity.setTitle(request.title());
        entity.setDescription(request.description());
        entity.setPriority(request.priority());
        entity.setCompleted(false); // Default value
        // user and category must be set by service layer
        return entity;
    }
    
    /**
     * Update existing TaskEntity with UpdateTaskRequest
     * Only updates non-null fields from request
     */
    public void updateEntity(TaskEntity entity, UpdateTaskRequest request) {
        if (request.title() != null) {
            entity.setTitle(request.title());
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
     * Convert TaskEntity to TaskResponse
     * Includes nested category information if present
     */
    public TaskResponse toResponse(TaskEntity entity) {
        TaskResponse.CategorySummary categoryInfo = null;
        
        // Map nested category
        if (entity.getCategory() != null) {
            CategoryEntity category = entity.getCategory();
            categoryInfo = new TaskResponse.CategorySummary(
                category.getId(),
                category.getName()
            );
        }
        
        return new TaskResponse(
            entity.getId(),
            entity.getTitle(),
            entity.getDescription(),
            entity.getCompleted(),
            entity.getPriority(),
            entity.getCreatedAt(),
            entity.getUpdatedAt(),
            categoryInfo
        );
    }
    
    /**
     * Convert TaskEntity to TaskSummary
     * Lighter version for list responses
     */
    public TaskSummary toSummary(TaskEntity entity) {
        return new TaskSummary(
            entity.getId(),
            entity.getTitle(),
            entity.getCompleted(),
            entity.getPriority(),
            entity.getCategory() != null ? entity.getCategory().getName() : null
        );
    }
    
    /**
     * Convert list of TaskEntity to list of TaskResponse
     */
    public List<TaskResponse> toResponseList(List<TaskEntity> entities) {
        return entities.stream()
            .map(this::toResponse)
            .collect(Collectors.toList());
    }
    
    /**
     * Convert list of TaskEntity to list of TaskSummary
     */
    public List<TaskSummary> toSummaryList(List<TaskEntity> entities) {
        return entities.stream()
            .map(this::toSummary)
            .collect(Collectors.toList());
    }
}
```

### Alternative: Using MapStruct

```java
package com.example.tasks.mapper;

import com.example.tasks.dto.*;
import com.example.tasks.entity.TaskEntity;
import org.mapstruct.*;

import java.util.List;

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface TaskMapperMapStruct {
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "user", ignore = true)
    @Mapping(target = "category", ignore = true)
    @Mapping(target = "completed", constant = "false")
    TaskEntity toEntity(CreateTaskRequest request);
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "user", ignore = true)
    @Mapping(target = "category", ignore = true)
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateEntity(@MappingTarget TaskEntity entity, UpdateTaskRequest request);
    
    @Mapping(target = "category", source = "category")
    TaskResponse toResponse(TaskEntity entity);
    
    @Mapping(target = "categoryName", source = "category.name")
    TaskSummary toSummary(TaskEntity entity);
    
    List<TaskResponse> toResponseList(List<TaskEntity> entities);
    
    List<TaskSummary> toSummaryList(List<TaskEntity> entities);
}
```

### Key Considerations

**Null Handling:**
Mappers must handle null values appropriately. For updates, null typically means "don't change this field."

**Record Accessors:**
Records use accessor methods without the "get" prefix. Use `request.title()` instead of `request.getTitle()`. Entity getters still use the traditional "get" prefix.

**Defensive Copying:**
When mapping collections, create new lists rather than exposing internal collections.

**Lazy Loading:**
Be cautious when mapping entities with lazy-loaded relationships. Access to lazy collections outside a transaction will cause LazyInitializationException.

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
package com.example.tasks.service;

import com.example.tasks.dto.*;
import com.example.tasks.entity.CategoryEntity;
import com.example.tasks.entity.TaskEntity;
import com.example.tasks.entity.UserEntity;
import com.example.tasks.exception.*;
import com.example.tasks.mapper.TaskMapper;
import com.example.tasks.repository.CategoryRepository;
import com.example.tasks.repository.TaskRepository;
import com.example.tasks.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.List;

@Service
@RequiredArgsConstructor
@Slf4j
public class TaskService {
    
    private final TaskRepository taskRepository;
    private final UserRepository userRepository;
    private final CategoryRepository categoryRepository;
    private final TaskMapper taskMapper;
    
    /**
     * Retrieve all tasks for a user
     * Read-only operation, no transaction needed
     */
    @Transactional(readOnly = true)
    public List<TaskSummary> getUserTasks(Long userId) {
        log.debug("Fetching tasks for user: {}", userId);
        
        // Verify user exists
        verifyUserExists(userId);
        
        List<TaskEntity> tasks = taskRepository.findByUserIdWithCategory(userId);
        return taskMapper.toSummaryList(tasks);
    }
    
    /**
     * Retrieve active tasks sorted by priority
     */
    @Transactional(readOnly = true)
    public List<TaskResponse> getActiveTasksByPriority(Long userId) {
        log.debug("Fetching active tasks by priority for user: {}", userId);
        
        verifyUserExists(userId);
        
        List<TaskEntity> tasks = taskRepository.findActiveTasksByPriority(userId);
        return taskMapper.toResponseList(tasks);
    }
    
    /**
     * Get a single task by ID
     */
    @Transactional(readOnly = true)
    public TaskResponse getTask(Long userId, Long taskId) {
        log.debug("Fetching task {} for user {}", taskId, userId);
        
        TaskEntity task = findTaskByIdAndUser(taskId, userId);
        return taskMapper.toResponse(task);
    }
    
    /**
     * Create a new task
     * Transactional - will rollback if any operation fails
     */
    @Transactional
    public TaskResponse createTask(Long userId, CreateTaskRequest request) {
        log.info("Creating task for user {}: {}", userId, request.title());
        
        // Verify user exists and get entity
        UserEntity user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + userId));
        
        // Map DTO to entity
        TaskEntity task = taskMapper.toEntity(request);
        task.setUser(user);
        
        // Set category if provided
        if (request.categoryId() != null) {
            CategoryEntity category = categoryRepository.findByIdAndUserId(
                request.categoryId(), userId
            ).orElseThrow(() -> new ResourceNotFoundException(
                "Category not found: " + request.categoryId()
            ));
            task.setCategory(category);
        }
        
        // Business rule: Check user's task limit
        long userTaskCount = taskRepository.countByUserIdAndCompleted(userId, false);
        if (userTaskCount >= 100) {
            throw new BusinessRuleException(
                "Cannot create task: user has reached maximum of 100 active tasks"
            );
        }
        
        // Save and return
        TaskEntity savedTask = taskRepository.save(task);
        log.info("Created task with ID: {}", savedTask.getId());
        
        return taskMapper.toResponse(savedTask);
    }
    
    /**
     * Update an existing task
     */
    @Transactional
    public TaskResponse updateTask(Long userId, Long taskId, UpdateTaskRequest request) {
        log.info("Updating task {} for user {}", taskId, userId);
        
        // Find existing task
        TaskEntity task = findTaskByIdAndUser(taskId, userId);
        
        // Update fields
        taskMapper.updateEntity(task, request);
        
        // Update category if provided
        if (request.categoryId() != null) {
            if (request.categoryId() == 0) {
                // Special case: 0 means remove category
                task.setCategory(null);
            } else {
                CategoryEntity category = categoryRepository.findByIdAndUserId(
                    request.categoryId(), userId
                ).orElseThrow(() -> new ResourceNotFoundException(
                    "Category not found: " + request.categoryId()
                ));
                task.setCategory(category);
            }
        }
        
        // Business rule: Cannot reopen urgent completed tasks after 30 days
        if (Boolean.FALSE.equals(request.completed()) && task.getCompleted()) {
            if (task.getPriority() == TaskPriority.URGENT) {
                LocalDateTime thirtyDaysAgo = LocalDateTime.now().minusDays(30);
                if (task.getUpdatedAt().isBefore(thirtyDaysAgo)) {
                    throw new BusinessRuleException(
                        "Cannot reopen urgent task that was completed more than 30 days ago"
                    );
                }
            }
        }
        
        TaskEntity savedTask = taskRepository.save(task);
        return taskMapper.toResponse(savedTask);
    }
    
    /**
     * Delete a task
     */
    @Transactional
    public void deleteTask(Long userId, Long taskId) {
        log.info("Deleting task {} for user {}", taskId, userId);
        
        TaskEntity task = findTaskByIdAndUser(taskId, userId);
        
        // Business rule: Cannot delete urgent tasks
        if (task.getPriority() == TaskPriority.URGENT && !task.getCompleted()) {
            throw new BusinessRuleException("Cannot delete active urgent tasks");
        }
        
        taskRepository.delete(task);
        log.info("Deleted task {}", taskId);
    }
    
    /**
     * Bulk delete all completed tasks
     */
    @Transactional
    public int deleteCompletedTasks(Long userId) {
        log.info("Deleting all completed tasks for user {}", userId);
        
        verifyUserExists(userId);
        
        List<TaskEntity> completedTasks = taskRepository.findByUserIdAndCompleted(userId, true);
        int count = completedTasks.size();
        
        taskRepository.deleteByUserIdAndCompleted(userId, true);
        
        log.info("Deleted {} completed tasks", count);
        return count;
    }
    
    /**
     * Get task statistics for a user
     */
    @Transactional(readOnly = true)
    public TaskStatistics getTaskStatistics(Long userId) {
        log.debug("Calculating task statistics for user {}", userId);
        
        verifyUserExists(userId);
        
        List<TaskEntity> allTasks = taskRepository.findByUserId(userId);
        long totalTasks = allTasks.size();
        long completedTasks = allTasks.stream().filter(TaskEntity::getCompleted).count();
        long activeTasks = totalTasks - completedTasks;
        
        LocalDateTime weekAgo = LocalDateTime.now().minusDays(7);
        List<TaskEntity> overdueTasks = taskRepository.findOverdueTasks(userId, weekAgo);
        
        return new TaskStatistics(totalTasks, activeTasks, completedTasks, overdueTasks.size());
    }
    
    // Private helper methods
    
    private void verifyUserExists(Long userId) {
        if (!userRepository.existsById(userId)) {
            throw new ResourceNotFoundException("User not found: " + userId);
        }
    }
    
    private TaskEntity findTaskByIdAndUser(Long taskId, Long userId) {
        return taskRepository.findByIdAndUserId(taskId, userId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Task not found: " + taskId + " for user: " + userId
            ));
    }
}
```

```java
package com.example.tasks.dto;

public record TaskStatistics(
    Long totalTasks,
    Long activeTasks,
    Long completedTasks,
    Long overdueTasks
) {}
```

### Key Considerations

**Transaction Management:**
Use `@Transactional` for operations that modify data. Use `@Transactional(readOnly = true)` for read operations to optimize performance.

**Business Rules:**
All business logic belongs in the service layer. This includes validation beyond simple format checks, complex calculations, and workflow enforcement.

**Error Handling:**
Services should throw domain-specific exceptions that controllers can translate into appropriate HTTP responses.

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
package com.example.tasks.controller;

import com.example.tasks.dto.*;
import com.example.tasks.service.TaskService;
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
@RequestMapping("/api/users/{userId}/tasks")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Tasks", description = "Task management endpoints")
public class TaskController {
    
    private final TaskService taskService;
    
    @GetMapping
    @Operation(summary = "Get all tasks for a user", description = "Retrieves all tasks for the specified user")
    @ApiResponse(responseCode = "200", description = "Tasks retrieved successfully")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<List<TaskSummary>> getUserTasks(
            @Parameter(description = "User ID") @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/tasks", userId);
        List<TaskSummary> tasks = taskService.getUserTasks(userId);
        return ResponseEntity.ok(tasks);
    }
    
    @GetMapping("/active")
    @Operation(summary = "Get active tasks by priority", 
               description = "Retrieves active tasks sorted by priority level")
    @ApiResponse(responseCode = "200", description = "Tasks retrieved successfully")
    public ResponseEntity<List<TaskResponse>> getActiveTasksByPriority(
            @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/tasks/active", userId);
        List<TaskResponse> tasks = taskService.getActiveTasksByPriority(userId);
        return ResponseEntity.ok(tasks);
    }
    
    @GetMapping("/{taskId}")
    @Operation(summary = "Get task by ID", description = "Retrieves a specific task by ID")
    @ApiResponse(responseCode = "200", description = "Task retrieved successfully")
    @ApiResponse(responseCode = "404", description = "Task not found")
    public ResponseEntity<TaskResponse> getTask(
            @PathVariable Long userId,
            @Parameter(description = "Task ID") @PathVariable Long taskId) {
        
        log.info("GET /api/users/{}/tasks/{}", userId, taskId);
        TaskResponse task = taskService.getTask(userId, taskId);
        return ResponseEntity.ok(task);
    }
    
    @PostMapping
    @Operation(summary = "Create a new task", description = "Creates a new task for the user")
    @ApiResponse(responseCode = "201", description = "Task created successfully")
    @ApiResponse(responseCode = "400", description = "Invalid request data")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<TaskResponse> createTask(
            @PathVariable Long userId,
            @Valid @RequestBody CreateTaskRequest request) {
        
        log.info("POST /api/users/{}/tasks - {}", userId, request.getTitle());
        TaskResponse task = taskService.createTask(userId, request);
        return ResponseEntity.status(HttpStatus.CREATED).body(task);
    }
    
    @PutMapping("/{taskId}")
    @Operation(summary = "Update a task", description = "Updates an existing task")
    @ApiResponse(responseCode = "200", description = "Task updated successfully")
    @ApiResponse(responseCode = "400", description = "Invalid request data")
    @ApiResponse(responseCode = "404", description = "Task not found")
    public ResponseEntity<TaskResponse> updateTask(
            @PathVariable Long userId,
            @PathVariable Long taskId,
            @Valid @RequestBody UpdateTaskRequest request) {
        
        log.info("PUT /api/users/{}/tasks/{}", userId, taskId);
        TaskResponse task = taskService.updateTask(userId, taskId, request);
        return ResponseEntity.ok(task);
    }
    
    @DeleteMapping("/{taskId}")
    @Operation(summary = "Delete a task", description = "Deletes a specific task")
    @ApiResponse(responseCode = "204", description = "Task deleted successfully")
    @ApiResponse(responseCode = "404", description = "Task not found")
    @ApiResponse(responseCode = "422", description = "Cannot delete urgent task")
    public ResponseEntity<Void> deleteTask(
            @PathVariable Long userId,
            @PathVariable Long taskId) {
        
        log.info("DELETE /api/users/{}/tasks/{}", userId, taskId);
        taskService.deleteTask(userId, taskId);
        return ResponseEntity.noContent().build();
    }
    
    @DeleteMapping("/completed")
    @Operation(summary = "Delete all completed tasks", 
               description = "Bulk deletes all completed tasks for the user")
    @ApiResponse(responseCode = "200", description = "Completed tasks deleted successfully")
    public ResponseEntity<DeleteCompletedResponse> deleteCompletedTasks(
            @PathVariable Long userId) {
        
        log.info("DELETE /api/users/{}/tasks/completed", userId);
        int deletedCount = taskService.deleteCompletedTasks(userId);
        return ResponseEntity.ok(new DeleteCompletedResponse(deletedCount));
    }
    
    @GetMapping("/statistics")
    @Operation(summary = "Get task statistics", 
               description = "Retrieves statistics about user's tasks")
    @ApiResponse(responseCode = "200", description = "Statistics retrieved successfully")
    public ResponseEntity<TaskStatistics> getTaskStatistics(
            @PathVariable Long userId) {
        
        log.info("GET /api/users/{}/tasks/statistics", userId);
        TaskStatistics stats = taskService.getTaskStatistics(userId);
        return ResponseEntity.ok(stats);
    }
}
```

```java
package com.example.tasks.dto;

public record DeleteCompletedResponse(int deletedCount) {}
```

### Global Exception Handler

```java
package com.example.tasks.exception;

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
Use `@Valid` to trigger validation on request DTOs. Let the framework handle validation errors.

**Path Variables:**
Use path variables for resource identifiers. Use query parameters for filters and pagination.

**Avoid:**
- Business logic in controllers
- Direct repository access
- Transaction management
- Complex data transformations
- Catching and hiding exceptions

---

## Data Flow

### Complete Request Flow Example

**Request:** `POST /api/users/1/tasks`

```json
{
  "title": "Complete documentation",
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
   - Triggers @Valid annotation validation on CreateTaskRequest
   - Validation passes
   - Calls: taskService.createTask(userId=1, request)

2. Service Layer
   - Receives userId and CreateTaskRequest DTO
   - Begins transaction (@Transactional)
   - Verifies user exists via userRepository.findById(1)
   - Calls mapper: taskMapper.toEntity(request)
   
3. Mapper Layer
   - Receives CreateTaskRequest DTO
   - Creates new TaskEntity
   - Copies: title, description, priority using record accessors (request.title(), etc.)
   - Sets completed = false
   - Returns TaskEntity (without user/category)
   
4. Service Layer (continued)
   - Receives TaskEntity from mapper
   - Sets user on entity via userRepository result
   - Finds category via categoryRepository.findByIdAndUserId(5, 1)
   - Sets category on entity
   - Checks business rule: user has < 100 active tasks
   - Business rule passes
   - Calls: taskRepository.save(entity)
   
5. Repository Layer
   - Receives TaskEntity
   - Generates SQL INSERT statement
   - Executes against database
   - Returns saved TaskEntity with generated ID
   
6. Service Layer (continued)
   - Receives saved TaskEntity from repository
   - Calls mapper: taskMapper.toResponse(entity)
   
7. Mapper Layer
   - Receives TaskEntity with full data
   - Creates TaskResponse DTO
   - Copies all fields including nested category
   - Returns TaskResponse
   
8. Service Layer (continued)
   - Returns TaskResponse to controller
   - Commits transaction
   
9. Controller Layer
   - Receives TaskResponse from service
   - Wraps in ResponseEntity with status 201 CREATED
   - Returns to client
   
10. Framework
    - Serializes TaskResponse to JSON
    - Sends HTTP response
```

**Response:** `201 Created`

```json
{
  "id": 42,
  "title": "Complete documentation",
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
public interface TaskRepository extends JpaRepository<TaskEntity, Long> {
    Page<TaskEntity> findByUserId(Long userId, Pageable pageable);
}

// Service
@Transactional(readOnly = true)
public Page<TaskSummary> getUserTasksPaginated(Long userId, int page, int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<TaskEntity> taskPage = taskRepository.findByUserId(userId, pageable);
    return taskPage.map(taskMapper::toSummary);
}

// Controller
@GetMapping
public ResponseEntity<Page<TaskSummary>> getUserTasks(
        @PathVariable Long userId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    
    Page<TaskSummary> tasks = taskService.getUserTasksPaginated(userId, page, size);
    return ResponseEntity.ok(tasks);
}
```

### 2. Filtering

```java
// Repository with Specification
public interface TaskRepository extends JpaRepository<TaskEntity, Long>, 
                                       JpaSpecificationExecutor<TaskEntity> {
}

// Specification builder
public class TaskSpecifications {
    
    public static Specification<TaskEntity> hasUserId(Long userId) {
        return (root, query, cb) -> cb.equal(root.get("user").get("id"), userId);
    }
    
    public static Specification<TaskEntity> hasCompleted(Boolean completed) {
        return (root, query, cb) -> cb.equal(root.get("completed"), completed);
    }
    
    public static Specification<TaskEntity> hasPriority(TaskPriority priority) {
        return (root, query, cb) -> cb.equal(root.get("priority"), priority);
    }
}

// Service
@Transactional(readOnly = true)
public List<TaskSummary> filterTasks(Long userId, Boolean completed, TaskPriority priority) {
    Specification<TaskEntity> spec = Specification.where(TaskSpecifications.hasUserId(userId));
    
    if (completed != null) {
        spec = spec.and(TaskSpecifications.hasCompleted(completed));
    }
    if (priority != null) {
        spec = spec.and(TaskSpecifications.hasPriority(priority));
    }
    
    List<TaskEntity> tasks = taskRepository.findAll(spec);
    return taskMapper.toSummaryList(tasks);
}
```

### 3. Soft Delete

```java
// Entity
@Entity
@Table(name = "tasks")
@SQLDelete(sql = "UPDATE tasks SET deleted_at = NOW() WHERE id = ?")
@Where(clause = "deleted_at IS NULL")
public class TaskEntity {
    // ... other fields ...
    
    @Column(name = "deleted_at")
    private LocalDateTime deletedAt;
}

// Service
@Transactional
public void deleteTask(Long userId, Long taskId) {
    TaskEntity task = findTaskByIdAndUser(taskId, userId);
    taskRepository.delete(task); // Triggers soft delete SQL
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
@Table(name = "tasks")
public class TaskEntity extends AuditableEntity {
    // ... task-specific fields ...
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
public class TaskEntity {
    private LocalDateTime dueDate;
    private Boolean completed;
    // Only getters/setters, no behavior
}

// Service does all the work
public class TaskService {
    public boolean isOverdue(TaskEntity task) {
        return !task.getCompleted() && 
               task.getDueDate().isBefore(LocalDateTime.now());
    }
}

// Better: Entity with domain methods
@Entity
public class TaskEntity {
    private LocalDateTime dueDate;
    private Boolean completed;
    
    public boolean isOverdue() {
        return !completed && dueDate.isBefore(LocalDateTime.now());
    }
    
    public void markComplete() {
        this.completed = true;
        this.updatedAt = LocalDateTime.now();
    }
}
```

### 2. Service Layer Bypass

Controllers should never directly access repositories.

```java
// Anti-pattern: Controller accesses repository directly
@RestController
public class TaskController {
    private final TaskRepository taskRepository;
    
    @GetMapping("/{id}")
    public TaskEntity getTask(@PathVariable Long id) {
        return taskRepository.findById(id).orElseThrow(); // Wrong!
    }
}

// Correct: Controller uses service
@RestController
public class TaskController {
    private final TaskService taskService;
    
    @GetMapping("/{id}")
    public TaskResponse getTask(@PathVariable Long id) {
        return taskService.getTask(id); // Right!
    }
}
```

### 3. Entity Exposure

Never expose entities directly in API responses.

```java
// Anti-pattern: Returning entity from controller
@GetMapping("/{id}")
public ResponseEntity<TaskEntity> getTask(@PathVariable Long id) {
    TaskEntity task = taskService.getTaskEntity(id); // Returns entity
    return ResponseEntity.ok(task); // Exposes internal structure!
}

// Correct: Return DTO (record)
@GetMapping("/{id}")
public ResponseEntity<TaskResponse> getTask(@PathVariable Long id) {
    TaskResponse task = taskService.getTask(id); // Returns DTO
    return ResponseEntity.ok(task);
}
```

### 4. Transaction Mismanagement

Avoid transactions in wrong layers.

```java
// Anti-pattern: Transaction in controller
@RestController
public class TaskController {
    
    @PostMapping
    @Transactional // Wrong layer!
    public TaskResponse createTask(@RequestBody CreateTaskRequest request) {
        // ...
    }
}

// Correct: Transaction in service
@Service
public class TaskService {
    
    @Transactional // Correct layer
    public TaskResponse createTask(CreateTaskRequest request) {
        // ...
    }
}
```

### 5. Business Logic in Mapper

Mappers should only transform data, not make decisions.

```java
// Anti-pattern: Business logic in mapper
public class TaskMapper {
    public TaskResponse toResponse(TaskEntity entity) {
        // Mapping code...
        TaskResponse response = new TaskResponse(...);
        
        // Business logic doesn't belong here!
        if (entity.getPriority() == TaskPriority.URGENT && !entity.getCompleted()) {
            // Cannot modify record after creation - would need builder pattern
            // This approach is fundamentally flawed with immutable DTOs
        }
        return response;
    }
}

// Correct: Keep mapper simple, handle display logic in service or presentation layer
public class TaskMapper {
    public TaskResponse toResponse(TaskEntity entity) {
        TaskResponse.CategorySummary categoryInfo = null;
        if (entity.getCategory() != null) {
            categoryInfo = new TaskResponse.CategorySummary(
                entity.getCategory().getId(),
                entity.getCategory().getName()
            );
        }
        
        // Pure transformation only
        return new TaskResponse(
            entity.getId(),
            entity.getTitle(),
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
class TaskRepositoryTest {
    
    @Autowired
    private TaskRepository taskRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void findByUserIdAndCompleted_ReturnsCorrectTasks() {
        // Given
        UserEntity user = createAndPersistUser();
        TaskEntity completedTask = createTask(user, true);
        TaskEntity activeTask = createTask(user, false);
        entityManager.persist(completedTask);
        entityManager.persist(activeTask);
        entityManager.flush();
        
        // When
        List<TaskEntity> completed = taskRepository.findByUserIdAndCompleted(
            user.getId(), true
        );
        
        // Then
        assertThat(completed).hasSize(1);
        assertThat(completed.get(0).getId()).isEqualTo(completedTask.getId());
    }
}
```

**Mapper Layer Testing:**

```java
class TaskMapperTest {
    
    private TaskMapper taskMapper = new TaskMapper();
    
    @Test
    void toEntity_MapsFieldsCorrectly() {
        // Given
        CreateTaskRequest request = new CreateTaskRequest(
            "Test Task",
            "Description",
            TaskPriority.HIGH,
            null
        );
        
        // When
        TaskEntity entity = taskMapper.toEntity(request);
        
        // Then
        assertThat(entity.getTitle()).isEqualTo("Test Task");
        assertThat(entity.getDescription()).isEqualTo("Description");
        assertThat(entity.getPriority()).isEqualTo(TaskPriority.HIGH);
        assertThat(entity.getCompleted()).isFalse();
        assertThat(entity.getId()).isNull(); // Not set by mapper
    }
    
    @Test
    void toResponse_HandlesNullCategory() {
        // Given
        TaskEntity entity = createTaskEntity();
        entity.setCategory(null);
        
        // When
        TaskResponse response = taskMapper.toResponse(entity);
        
        // Then
        assertThat(response.category()).isNull();
    }
}
```

**Service Layer Testing:**

```java
@ExtendWith(MockitoExtension.class)
class TaskServiceTest {
    
    @Mock
    private TaskRepository taskRepository;
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private CategoryRepository categoryRepository;
    
    @Mock
    private TaskMapper taskMapper;
    
    @InjectMocks
    private TaskService taskService;
    
    @Test
    void createTask_Success() {
        // Given
        Long userId = 1L;
        CreateTaskRequest request = new CreateTaskRequest(
            "Test",
            null,
            TaskPriority.MEDIUM,
            null
        );
        
        UserEntity user = new UserEntity();
        user.setId(userId);
        
        TaskEntity entity = new TaskEntity();
        TaskEntity savedEntity = new TaskEntity();
        savedEntity.setId(1L);
        
        TaskResponse expected = new TaskResponse(
            1L, "Test", null, false, TaskPriority.MEDIUM,
            LocalDateTime.now(), LocalDateTime.now(), null
        );
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(taskMapper.toEntity(request)).thenReturn(entity);
        when(taskRepository.countByUserIdAndCompleted(userId, false)).thenReturn(50L);
        when(taskRepository.save(entity)).thenReturn(savedEntity);
        when(taskMapper.toResponse(savedEntity)).thenReturn(expected);
        
        // When
        TaskResponse result = taskService.createTask(userId, request);
        
        // Then
        assertThat(result.id()).isEqualTo(1L);
        verify(taskRepository).save(entity);
        verify(taskMapper).toResponse(savedEntity);
    }
    
    @Test
    void createTask_ExceedsLimit_ThrowsException() {
        // Given
        Long userId = 1L;
        CreateTaskRequest request = new CreateTaskRequest(
            "Test",
            null,
            TaskPriority.MEDIUM,
            null
        );
        
        UserEntity user = new UserEntity();
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(taskRepository.countByUserIdAndCompleted(userId, false)).thenReturn(100L);
        
        // When/Then
        assertThatThrownBy(() -> taskService.createTask(userId, request))
            .isInstanceOf(BusinessRuleException.class)
            .hasMessageContaining("maximum of 100 active tasks");
        
        verify(taskRepository, never()).save(any());
    }
}
```

**Controller Layer Testing:**

```java
@WebMvcTest(TaskController.class)
class TaskControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private TaskService taskService;
    
    @Test
    void createTask_ValidRequest_Returns201() throws Exception {
        // Given
        Long userId = 1L;
        
        TaskResponse response = new TaskResponse(
            1L,
            "Test Task",
            null,
            false,
            TaskPriority.HIGH,
            LocalDateTime.now(),
            LocalDateTime.now(),
            null
        );
        
        when(taskService.createTask(eq(userId), any(CreateTaskRequest.class)))
            .thenReturn(response);
        
        // When/Then
        mockMvc.perform(post("/api/users/{userId}/tasks", userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "title": "Test Task",
                        "priority": "HIGH"
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.title").value("Test Task"));
    }
    
    @Test
    void createTask_InvalidRequest_Returns400() throws Exception {
        // When/Then
        mockMvc.perform(post("/api/users/{userId}/tasks", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "title": "",
                        "priority": "INVALID"
                    }
                    """))
            .andExpect(status().isBadRequest());
        
        verifyNoInteractions(taskService);
    }
}
```

---

## Performance Considerations

### N+1 Query Problem

```java
// Problem: N+1 queries
@Transactional(readOnly = true)
public List<TaskResponse> getTasks(Long userId) {
    List<TaskEntity> tasks = taskRepository.findByUserId(userId); // 1 query
    
    return tasks.stream()
        .map(task -> {
            task.getCategory().getName(); // N queries (lazy loading)
            return taskMapper.toResponse(task);
        })
        .collect(Collectors.toList());
}

// Solution: Use JOIN FETCH
@Query("SELECT t FROM TaskEntity t LEFT JOIN FETCH t.category WHERE t.user.id = :userId")
List<TaskEntity> findByUserIdWithCategory(@Param("userId") Long userId);
```

### Batch Operations

```java
// Inefficient: Individual saves
public void createMultipleTasks(List<CreateTaskRequest> requests) {
    for (CreateTaskRequest request : requests) {
        TaskEntity entity = taskMapper.toEntity(request);
        taskRepository.save(entity); // Individual INSERT
    }
}

// Better: Batch save
public void createMultipleTasks(List<CreateTaskRequest> requests) {
    List<TaskEntity> entities = requests.stream()
        .map(taskMapper::toEntity)
        .collect(Collectors.toList());
    
    taskRepository.saveAll(entities); // Batch INSERT
}
```

### DTO Projection

```java
// Inefficient: Load full entities for summary
public List<TaskSummary> getTaskSummaries(Long userId) {
    List<TaskEntity> tasks = taskRepository.findByUserId(userId); // Full entities
    return taskMapper.toSummaryList(tasks);
}

// Better: Use projection
public interface TaskSummaryProjection {
    Long getId();
    String getTitle();
    Boolean getCompleted();
    TaskPriority getPriority();
}

@Query("SELECT t.id as id, t.title as title, t.completed as completed, " +
       "t.priority as priority FROM TaskEntity t WHERE t.user.id = :userId")
List<TaskSummaryProjection> findSummariesByUserId(@Param("userId") Long userId);
```

### Caching

```java
@Service
public class TaskService {
    
    // Cache user task counts
    @Cacheable(value = "taskCounts", key = "#userId")
    @Transactional(readOnly = true)
    public Long getUserTaskCount(Long userId) {
        return taskRepository.countByUserId(userId);
    }
    
    // Evict cache on modifications
    @CacheEvict(value = "taskCounts", key = "#userId")
    @Transactional
    public TaskResponse createTask(Long userId, CreateTaskRequest request) {
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
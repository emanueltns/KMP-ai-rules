# KMP Architecture Patterns Documentation

## Table of Contents
1. [Offline-First Architecture Pattern](#offline-first-architecture-pattern)
2. [Gateway Implementation Pattern](#gateway-implementation-pattern)
3. [Operation Type Pattern](#operation-type-pattern)
4. [Database Integration Pattern](#database-integration-pattern)
5. [Gateway Quality Checks](#gateway-quality-checks)

# Offline-First Architecture Pattern

## Core Components

### 1. OfflineController
```kotlin
class OfflineController(
    private val responseRuleSetter: ResponseRuleSetter
): KoinComponent {
    private val _isOnline = MutableStateFlow(true)
    val isOnline: StateFlow<Boolean> = _isOnline
    // ...
}
```

Key Features:
- Network state management using StateFlow
- Dependency injection with Koin
- Polymorphic serialization support for models
- Sync operation management

### 2. Operation Types
```kotlin
@Serializable
sealed class OperationType {
    interface OperationProperties {
        val requestType: RequestType
        val url: String
        val body: String?
    }
    // Operation categories (Route, Message, Dashboard)
}
```

Features:
- Serializable operations for offline storage
- Type-safe operation properties
- Hierarchical operation structure
- Domain-specific operation categories

### 3. Sync Operation Management
```kotlin
data class SyncOperation(
    val id: String,
    val requestType: RequestType,
    val operationType: OperationType,
    val url: String,
    val body: String?,
    val timestamp: Long,
    val priority: Int
)
```

## Implementation Rules

1. **Network State Management**
   - Use StateFlow for online/offline status
   - Provide state update mechanisms
   - Expose read-only state to consumers

2. **Operation Handling**
   - Define operations as sealed classes
   - Include all necessary metadata
   - Support serialization for storage
   - Implement priority-based sync

3. **Data Persistence**
   - Store operations in local database
   - Generate unique IDs for sync operations
   - Maintain operation timestamps
   - Support priority-based queuing

4. **Response Simulation**
   - Implement ResponseRuleSetter for offline mode
   - Support polymorphic model serialization
   - Provide type-safe response generation

## Quality Checks

1. Network State:
   - Is state properly initialized?
   - Are state updates thread-safe?
   - Is state observable throughout the app?

2. Operation Types:
   - Are all operations properly serializable?
   - Do operations contain required metadata?
   - Is the operation hierarchy logical?

3. Sync Management:
   - Is operation storage reliable?
   - Are IDs truly unique?
   - Is priority handling correct?

## Usage Example
```kotlin
class Gateway(private val offlineController: OfflineController) {
    suspend fun execute() = withContext(dispatchers.io) {
        if (!offlineController.isOnline.value) {
            handleOfflineMode()
        }
    }
}
```

# Gateway Implementation Pattern

## Core Components

### 1. Base Gateway Structure
```kotlin
abstract class BaseGateway(
    val client: HttpClient,
) : IGateway, KoinComponent {
    val offlineController: OfflineController by inject()
    val database: AppDatabase by inject()
}
```

Key Features:
- Abstract base class for all gateways
- Dependency injection with Koin
- HTTP client integration
- Database access

### 2. Type-Safe Request Configuration
```kotlin
protected data class RequestConfig(
    val baseUrl: String,
    val params: Map<String, String> = emptyMap(),
    val body: String? = null
)
```

Features:
- Immutable request configuration
- Type-safe parameter mapping
- Optional request body
- URL parameter handling

### 3. Error Handling and Execution
```kotlin
suspend inline fun <reified T> tryToExecute(
    method: HttpClient.() -> HttpResponse, 
    operationType: OperationType
): T {
    if (!offlineController.isOnline.value) {
        return handleOfflineMode<T>(operationType)
    }
    // Error handling and response processing
}
```

## Implementation Rules

1. **Interface-First Design**
   - Define gateway interfaces (IGateway)
   - Implement concrete gateways
   - Support mock implementations for testing

2. **Type Safety**
   - Use generics for response types
   - Implement type-safe request builders
   - Ensure null-safety in responses

3. **Error Handling**
   - Standardized error responses
   - Offline mode handling
   - Network error management

4. **Configuration Management**
   - Type-safe request configuration
   - URL parameter handling
   - Session token management

## Quality Checks

1. Request Configuration:
   - Are all parameters properly typed?
   - Is URL construction safe?
   - Are optional parameters handled?

2. Error Handling:
   - Are all error cases covered?
   - Is offline mode properly handled?
   - Are exceptions properly typed?

3. Type Safety:
   - Are generics used correctly?
   - Is null-safety enforced?
   - Are responses properly typed?

## Usage Example
```kotlin
class RouteGateway : BaseGateway {
    suspend fun loadRoute(id: Long): RouteModel? {
        val config = RequestConfig(
            baseUrl = "$ROUTING$LOAD_ROUTE",
            params = mapOf(
                PARAM_MOBILE_DEVICE_ID to mobileDeviceId,
                PARAM_ROUTE_ID to id.toString()
            )
        )
        return executeWithConfig(
            config = config,
            method = { url -> client.post(url) },
            operationBuilder = { url, body ->
                OperationType.RouteOperation.LoadRoute(...)
            }
        )
    }
}
```

# Operation Type Pattern

## Core Structure

### 1. Base Operation Type
```kotlin
@Serializable
sealed class OperationType {
    interface OperationProperties {
        val requestType: RequestType
        val url: String
        val body: String?
    }
}
```

Key Features:
- Serializable for offline storage
- Interface-based property definition
- Common properties for all operations

### 2. Domain-Specific Operations
```kotlin
@Serializable
sealed class RouteOperation : OperationType(), OperationProperties {
    @Serializable
    data class LoadRoute(
        override val requestType: RequestType,
        override val url: String,
        override val body: String?,
        val routeId: Long
    ) : RouteOperation()
}
```

Features:
- Domain separation (Route, Message, Dashboard, Activity)
- Type-safe operation parameters
- Operation-specific properties

## Implementation Rules

1. **Serialization Support**
   - All operations must be @Serializable
   - Support polymorphic serialization
   - Handle nullable properties

2. **Type Hierarchy**
   - Base sealed class for all operations
   - Domain-specific sealed classes
   - Concrete data classes for operations

3. **Property Requirements**
   - Common properties through interface
   - Operation-specific properties
   - Type-safe property definitions

4. **Domain Organization**
   - Group related operations
   - Maintain operation isolation
   - Support extensibility

# Database Integration Pattern

## Core Components

### 1. SQLDelight Integration
```kotlin
expect class DatabaseDriverFactory() {
    fun createDatabaseDriver(schema: SqlSchema<QueryResult.AsyncValue<Unit>>): SqlDriver
}
```

Key Features:
- Platform-specific database drivers
- Asynchronous query support
- Schema-based database creation

### 2. Schema Definition
```sql
-- Example: Offline Action Schema
CREATE TABLE IF NOT EXISTS OfflineAction (
    id TEXT PRIMARY KEY NOT NULL,
    operation_type TEXT NOT NULL,
    request_type TEXT NOT NULL,
    url TEXT NOT NULL,
    body TEXT,
    timestamp INTEGER NOT NULL,
    priority INTEGER NOT NULL
);
```

Features:
- Type-safe schema definitions
- Platform-independent SQL
- Clear table structures

### 3. Query Definitions
```sql
-- Example: Session Queries
insertSession:
INSERT OR REPLACE INTO SessionDao VALUES (?,...);

getSession:
SELECT * FROM SessionDao LIMIT 1;

deleteSession:
DELETE FROM SessionDao;
```

## Implementation Rules

1. **Schema Management**
   - Use SQLDelight for type-safe SQL
   - Define clear table structures
   - Maintain schema versions

2. **Query Organization**
   - Group related queries
   - Use meaningful query names
   - Support transaction handling

3. **Platform Specifics**
   - Abstract driver creation
   - Handle platform differences
   - Support async operations

4. **Data Access**
   - Type-safe query results
   - Transaction support
   - Error handling

# Gateway Quality Checks

## Core Quality Areas

### 1. Architecture Compliance
```kotlin
abstract class BaseGateway(
    val client: HttpClient,
) : IGateway, KoinComponent {
    val offlineController: OfflineController by inject()
    val database: AppDatabase by inject()
}
```

Checks:
- Interface implementation
- Dependency injection usage
- Component responsibilities

### 2. Error Handling
```kotlin
suspend inline fun <reified T> tryToExecute(
    method: HttpClient.() -> HttpResponse, 
    operationType: OperationType
): T {
    if (!offlineController.isOnline.value) {
        return handleOfflineMode<T>(operationType)
    }
    try {
        // Network call
    } catch (e: Exception) {
        // Error handling
    }
}
```

Checks:
- Error type coverage
- Recovery mechanisms
- Error propagation

### 3. Type Safety
```kotlin
protected data class RequestConfig(
    val baseUrl: String,
    val params: Map<String, String>,
    val body: String?
)
```

Checks:
- Generic type usage
- Null safety
- Type constraints

## Quality Check Categories

1. **Architecture Checks**
   - Clean Architecture compliance
   - Interface segregation
   - Dependency management
   - Layer separation

2. **Implementation Checks**
   - Error handling completeness
   - Type safety enforcement
   - Resource management
   - Memory leaks prevention

3. **Performance Checks**
   - Connection pooling
   - Request caching
   - Resource cleanup
   - Memory usage

4. **Security Checks**
   - Authentication handling
   - Data encryption
   - Token management
   - Input validation

## Testing Requirements

1. **Unit Tests**
   ```kotlin
   @Test
   fun `test gateway offline behavior`() {
       // Given
       offlineController.updateNetworkState(false)
       
       // When
       val result = gateway.loadRoute(1L)
       
       // Then
       verify(offlineController).performOperation(any())
   }
   ```

2. **Integration Tests**
   ```kotlin
   @Test
   fun `test complete request flow`() {
       // Test network -> offline -> network transition
   }
   ```

## Best Practices

1. **Code Organization**
   - Clear responsibility separation
   - Consistent error handling
   - Proper documentation
   - Clean code structure

2. **Resource Management**
   - Connection pooling
   - Memory management
   - Cache control
   - Cleanup routines

3. **Error Handling**
   - Comprehensive error types
   - Recovery mechanisms
   - User feedback
   - Logging strategy

4. **Testing Strategy**
   - Unit test coverage
   - Integration testing
   - Error scenario testing
   - Performance testing

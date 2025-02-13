# Property Type Modification Rules

## Overview
This document outlines the rules and best practices for modifying property types in the MobileAssist-KMP codebase, especially when the properties are used across multiple files and components.

## 1. Property Usage Analysis
Before modifying any property type:
- [ ] Run a codebase-wide search for all usages
- [ ] Document all files and components that use this property
- [ ] Identify all interfaces that expose this property
- [ ] Check for serialization/deserialization usage

## 2. Type Modification Checklist
Example of property with multiple usages:
```kotlin
data class Route(
    val travelTime: Duration,  // Used in multiple places
    val stops: List<Stop>
)
```

Before changing any property type, verify:
- [ ] Is this property used in API responses?
- [ ] Is this property used in database entities?
- [ ] Is this property used in UI displays?
- [ ] Are there any mappers using this property?
- [ ] Is this property part of an interface contract?

## 3. Safe Modification Strategy
```kotlin
// CORRECT: Phased approach
// Phase 1: Add new property while keeping old
data class Route(
    @Deprecated("Use travelDuration instead")
    val travelTime: Int,
    val travelDuration: Duration
)

// Phase 2: Migrate all usages
// Phase 3: Remove deprecated property
```

## 4. Breaking Change Prevention
- Create interface contracts for shared models
- Use sealed interfaces for type-safe hierarchies
- Document property types in interface comments
- Add @JvmInline value classes for type safety

## 5. Testing Requirements
- Unit tests for all property access patterns
- Integration tests for data flow
- Serialization/deserialization tests
- Migration tests for database changes

## 6. Documentation Requirements
```kotlin
/**
 * @property travelTime Duration of travel
 * @warning This property is used in:
 *          - DashboardRepository (display)
 *          - RouteGateway (API)
 *          - RouteEntity (Database)
 */
```

## 7. Common Anti-patterns to Avoid
```kotlin
// DON'T: Direct type changes without migration
val travelTime: Int  // -> Duration (breaks existing code)

// DON'T: Implicit type conversions
val time: Duration = someInt // Type mismatch

// DO: Explicit conversions with migration path
val travelTime: Duration = someInt.toDuration(DurationUnit.MINUTES)
```

## 8. Review Process
Before merging type changes:
- [ ] All usages updated
- [ ] Tests passing
- [ ] Migration strategy documented
- [ ] Breaking changes communicated
- [ ] Rollback plan in place

## Best Practices Summary
1. Always analyze property usage before modification
2. Use a phased approach for type changes
3. Keep comprehensive documentation
4. Ensure thorough testing coverage
5. Follow the review process checklist

## Contributing
Feel free to suggest improvements to these rules by creating a pull request or discussing in the team channel.

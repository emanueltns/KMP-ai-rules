# Comprehensive Code Search and Update Strategy

## Directory Structure Analysis
When modifying code, systematically check these directories:
```
/src/
  ├── commonMain/
  │   ├── domain/models/       # Domain models
  │   ├── data/
  │   │   ├── models/         # DTOs and Entities
  │   │   ├── gateway/
  │   │   │   ├── local/
  │   │   │   │   └── mockData/  # Mock implementations
  │   │   │   └── remote/    # API implementations
  │   │   └── mapper/        # Model mappers
  │   └── test/             # Test implementations
  └── test/
      └── kotlin/           # Platform-specific tests
```

## Search Strategies
1. **Type-based Search**
   ```
   - Search by class name: "ClassDto", "ClassEntity"
   - Search by interface name: "IClass"
   - Search by property names: "propertyName"
   ```

2. **File Pattern Search**
   ```
   - Mock data: "*Data.kt", "*Mock*.kt"
   - Tests: "*Test.kt", "*Spec.kt"
   - Fakes: "*Fake*.kt", "*Stub*.kt"
   ```

3. **Directory-specific Search**
   ```
   - /mockData/ - Mock implementations
   - /test/ - Test implementations
   - /fake/ or /stub/ - Test doubles
   - /fixtures/ - Test fixtures
   ```

## Quality Checklist
Before completing any code modification:
- [ ] Domain models checked and updated
- [ ] DTOs and Entities checked and updated
- [ ] Mappers checked and updated
- [ ] Mock data files checked and updated
- [ ] Test data files checked and updated
- [ ] Fake/stub implementations checked and updated
- [ ] Test fixtures checked and updated

## Common Locations for Test Data
1. Mock Data:
   - Local gateway mock data
   - API response mocks
   - Database fixtures

2. Test Implementations:
   - Unit test helper classes
   - Integration test data
   - UI test fixtures

3. Demo/Sample Data:
   - Demo mode implementations
   - Sample data for development
   - Documentation examples

## Best Practices
1. **Search Order**
   - Start with direct usages
   - Check test implementations
   - Check mock/fake data
   - Check documentation examples

2. **Update Strategy**
   - Update production code first
   - Update tests and mocks to match
   - Update documentation and examples
   - Verify all changes with tests

3. **Validation**
   - Run all tests after updates
   - Check demo mode functionality
   - Verify documentation accuracy
   - Test with sample data

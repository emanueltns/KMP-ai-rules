val global_rules = """
Analyze the following code request and generate appropriate Kotlin Multiplatform implementations following these rules:

<critical_rules>
1. NO_CODE_MODIFICATION: Never modify existing code without explicit permission
2. FAIL_FAST: Immediately report and fix compile/runtime errors
3. ERROR_LOGGING: Maintain error registry for future prevention
4. INTERFACE_FIRST: Prefer interface declarations over implementations
5. USE EXISTING ARCHITECTURE: DO NOT use rules from <architecture_guidlines> which are not apply to existing architecture. 
6. UPDATE RULES: Whenever you find something that can be added into the rules, update document is you have access or create a new one and write there the rules. Follow the same tamplet like in this document.
7. RULE OUTPUT: Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.
</critical_rules>

<architecture_guidelines>
1. Dependency Management:
   - Use dependency injection with Koin
   - Follow "single" instance pattern for core services
   - Apply dependency inversion for platform-specific code

2. Hierarchy Rules:
   High-level modules must depend on abstractions:
   commonMain -> platformInterfaces
   platformSpecific <- platformImplementations

3. Code Structure:
   - Single Responsibility principle for all components
   - Coroutine-aware implementations
   - Platform-agnostic domain logic
   - ViewModels delegate to UseCases in commonMain

4. Testability:
   - Mockable interfaces for all dependencies
   - Test implementations in corresponding test directories
</architecture_guidelines>

<clean_architecture>
1. Dependency Direction:
   Presentation → Domain ← Data ← Platform
2. Layer Responsibilities:
   - Presentation: UI/ViewModel (platform-agnostic)
   - Domain: Entities, UseCases, Interfaces
   - Data: Repositories, DataSources 
   - Platform: Network/DB implementations
</clean_architecture>

<quality_checks>
Check code against these criteria before output:
1. Does this introduce platform-specific code in commonMain?
2. Are we exposing implementation details through interfaces?
3. Have we maintained proper dependency direction?
4. Does this violate any SOLID principles?
5. Are we following Kotlin coding conventions?
6. Are dependencies properly version-managed?
7. Are there any potential memory leaks?
8. Are concurrency patterns correct?
9. Are we using correct property accessors for each type?
10. Have we checked API documentation for correct property names?
11. Are we using type-specific conversion methods?
12. Is there proper null handling in conversions?
13. Are you using the correct type-specific properties?
14. Did you analyze property usage before modification?
</quality_checks>

<key_enhancements_for_clean_architecture_in_KMM>
1. **Strict Layer Separation:** Enforce unidirectional dependencies between features→domain←data←platform
2. **Use Case Centric Design:** Core business logic isolated in domain layer
3. **Platform-Agnostic Domain:** Pure Kotlin code shared across all targets
4. **Validation Annotations:** Compile-time checks through custom annotations
5. **Coroutine Context Enforcement:** Prevent main thread blocking
6. **Standardized Error Handling:** Unified Result pattern across layers
7. **DI Layer Isolation:** Separate module definitions per architecture layer
8. **Cross-Platform ViewModels:** Common presentation logic in shared modules
</key_enhancements_for_clean_architecture_in_KMM>


<additional_safeguards>
- **File Path Validation:** Compiler checks for `domain` in `commonMain/domain`
- **Null Safety Propagation:** Domain models are non-nullable by default
- **DTO Separation:** Platform-specific data transfer objects never leak to domain
- **Single Activity Direction:** Features can't directly access platform layer
- **Test-Specific Rules:** Mock implementations only in `test/` source sets
</additional_safeguards>

<datetime_handling_rules>
1. LocalDateTime Operations:
   - ALWAYS use .date property when performing arithmetic operations
   - NEVER perform direct minus/plus on LocalDateTime objects
   - Use kotlinx.datetime functions instead of platform-specific implementations

2. Duration Calculations:
   - ALWAYS specify explicit DurationUnit
   - Convert time differences to Duration using .toDuration()
   - Handle null cases with Duration.ZERO as default

3. Time Comparison Pattern:
   ```kotlin
   // CORRECT
   val duration = endTime.date.minus(startTime.date).toMillis().toDuration(DurationUnit.MILLISECONDS)
   
   // INCORRECT
   val duration = endTime.minus(startTime).toMillis()
   
4. Time Safety Checklist:
		[ ] Are we accessing .date property for arithmetic?
		[ ] Is DurationUnit explicitly specified?
		[ ] Are null cases handled with safe defaults?
		[ ] Are we using kotlinx.datetime extensions?
		
5. Common Patterns:
    // Duration calculation pattern
		val duration = when {
    start != null && end != null ->
        end.date.minus(start.date).toMillis().toDuration(DurationUnit.MILLISECONDS)
    else -> Duration.ZERO
		}

		// Time accumulation pattern
		var totalDuration = Duration.ZERO
		items.forEach { item ->
		    if (item.start != null && item.end != null) {
		        totalDuration += item.end.date.minus(item.start.date)
		            .toMillis()
		            .toDuration(DurationUnit.MILLISECONDS)
		    }
		}

6. Best Practices:
		Use explicit type declarations for Duration properties
		Prefer functional operations over imperative loops for time calculations
		Always provide safe fallbacks for null time values
		Use extension functions for common time operations 
   
</datetime_handling_rules>   

<output_format>
Generate implementations with:
1. Exact file paths (including platform dirs)
2. Rule annotations (e.g., // [INTERFACE_FIRST])
3. Change markers when modifying existing code:
   - // ... existing code ...
   - {{ implemented_component }}
   - // ... existing code ...
4. Compiler-safe code (fully typed with generics)
5. Null safety (strict null checking)
6. Type-safe platform-specific implementations
</output_format>

<incomplete_request_handling>
If any of these conditions exist:
a) Missing architecture requirements
b) Unclear platform-specific requirements
c) Ambiguous business logic
Return structured response:
{
  "status": "NEED_CLARIFICATION",
  "required_info": ["list of missing elements"]
}
</incomplete_request_handling>

<examples>
1. When creating a network service:
// [SINGLE_INSTANCE, INTERFACE_BINDING]
val networkModule = module {
    single<NetworkService> { KtorNetworkService() }
    single<NetworkServiceInterface> { get<NetworkService>() }
}

2. Error handling pattern:
// [ERROR_LOGGING, DRY]
class ErrorRepository {
    private val errorRegistry = mutableMapOf<String, String>()
    
    fun logError(context: String, error: Throwable) {
        errorRegistry[context] = error.stackTraceToString()
        // Platform-specific crash reporting
        expect fun reportToCrashlytics(error: Throwable)
    }
}
</examples>


<property_type_rules>
		<property_usage_analysis>
        <checklist>
            <item>Run a codebase-wide search for all usages</item>
            <item>Document all files and components that use this property</item>
            <item>Identify all interfaces that expose this property</item>
            <item>Check for serialization/deserialization usage</item>
        </checklist>
    </property_usage_analysis>
    
    <type_modification_checklist>
        <example language="kotlin">
            <code>
                data class Route(
                    val travelTime: Duration,  // Used in multiple places
                    val stops: List<Stop>
                )
            </code>
        </example>
        <verification_points>
            <point>Is this property used in API responses?</point>
            <point>Is this property used in database entities?</point>
            <point>Is this property used in UI displays?</point>
            <point>Are there any mappers using this property?</point>
            <point>Is this property part of an interface contract?</point>
        </verification_points>
    </type_modification_checklist>
    
    <safe_modification_strategy>
        <example language="kotlin">
            <code>
                // CORRECT: Phased approach
                // Phase 1: Add new property while keeping old
                data class Route(
                    @Deprecated("Use travelDuration instead")
                    val travelTime: Int,
                    val travelDuration: Duration
                )
                
                // Phase 2: Migrate all usages
                // Phase 3: Remove deprecated property
            </code>
        </example>
    </safe_modification_strategy>
    
    <breaking_change_prevention>
        <guidelines>
            <item>Create interface contracts for shared models</item>
            <item>Use sealed interfaces for type-safe hierarchies</item>
            <item>Document property types in interface comments</item>
            <item>Add @JvmInline value classes for type safety</item>
        </guidelines>
    </breaking_change_prevention>

    <testing_requirements>
        <requirements>
            <requirement>Unit tests for all property access patterns</requirement>
            <requirement>Integration tests for data flow</requirement>
            <requirement>Serialization/deserialization tests</requirement>
            <requirement>Migration tests for database changes</requirement>
        </requirements>
    </testing_requirements>
    
    <documentation_requirements>
        <example language="kotlin">
            <code>
                /**
                 * @property travelTime Duration of travel
                 * @warning This property is used in:
                 *          - DashboardRepository (display)
                 *          - RouteGateway (API)
                 *          - RouteEntity (Database)
                 **/
            </code>
        </example>
    </documentation_requirements>
    
    <anti_patterns>
        <example language="kotlin">
            <code>
                // DON'T: Direct type changes without migration
                val travelTime: Int  // -> Duration (breaks existing code)

                // DON'T: Implicit type conversions
                val time: Duration = someInt // Type mismatch

                // DO: Explicit conversions with migration path
                val travelTime: Duration = someInt.toDuration(DurationUnit.MINUTES)
            </code>
        </example>
    </anti_patterns>
</property_type_rules>



"""

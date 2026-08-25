---
id: PP-20260825-7ad4b1
title: "Externalized Jackson serialization — no behavior annotations on domain types"
type: principle
scope: platform
applies_to: "All casehub domain model types that participate in serialization"
severity: important
refs:
  - casehub/engine/api/src/main/java/io/casehub/api/model/CaseDefinition.java
  - casehub/engine/api/src/main/java/io/casehub/api/model/converter/CaseDefinitionYamlMapper.java
violation_hint: "Adding @JsonCreator, @JsonDeserialize, @JsonTypeInfo, or @JsonSubTypes to a domain model class"
created: 2026-08-25
---

Domain model types must not carry Jackson behavior annotations. Serialization
and deserialization rules are externalized via Jackson Modules, keeping domain
types independent of their serialization format.

**Metadata annotations are fine** on domain types:
- `@JsonPropertyDescription` — documents the field; used by schema generators
- `@JsonIgnore` — marks runtime-only fields that have no serialized form
- `@JsonProperty` for simple name mapping when the Java name matches intent

**Behavior annotations must be externalized:**
- `@JsonCreator`, `@JsonDeserialize`, `@JsonSerialize` — control construction and conversion
- `@JsonTypeInfo`, `@JsonSubTypes` — control polymorphic dispatch
- `@JsonUnwrapped`, `@JsonAnySetter`, `@JsonAnyGetter` — control structural mapping
- Custom deserializer registration

**Where the rules live:** A Jackson `Module` subclass in the converter/serialization
package of the owning module. Registered on the `ObjectMapper` at the call site —
never via global configuration.

```java
// Domain type — clean
public class CaseDefinition {
    @JsonPropertyDescription("Unique namespace for this case definition")
    private String namespace;

    @JsonIgnore  // runtime-only — no serialized form
    private ContextBridge<?> defaultWorkerBridge;
}

// Externalized — in the converter package
public class CaseDefinitionModule extends SimpleModule {
    public CaseDefinitionModule() {
        addDeserializer(Trigger.class, new TriggerDeserializer());
        addDeserializer(BindingTarget.class, new BindingTargetDeserializer());
    }
}
```

**Rationale:** Domain types are consumed across repos, serialization formats, and
contexts. Coupling them to Jackson forces every consumer to depend on Jackson's
type system, prevents alternative serialization strategies, and mixes structural
concerns (what the type IS) with infrastructure concerns (how it's transported).
The externalized module pattern keeps the domain pure and the serialization swappable.

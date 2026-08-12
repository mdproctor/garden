---
id: PP-20260812-2dab38
title: "orm.xml tier separation for api/ purity"
type: rule
scope: platform
applies_to: "casehub foundation repos — api/ modules with domain model classes requiring JPA mapping"
severity: critical
refs:
  - ../../casehub/platform-api-scope.md
  - ../../universal/module-tier-structure.md
violation_hint: "jakarta.persistence-api in api/ module pom.xml, or @MappedSuperclass/@Column annotations on classes in api/ packages"
created: 2026-08-12
---

When a foundation repo's api/ module contains domain model classes that require JPA mapping (e.g. @MappedSuperclass fields), declare the mappings in the runtime module's `META-INF/orm.xml` as `<mapped-superclass>` elements rather than annotating the api/ classes directly. This keeps api/ zero-dependency (no jakarta.persistence-api) while preserving the inheritance hierarchy (runtime @Entity classes extend api model classes). The orm.xml is discovered automatically by Quarkus Hibernate ORM from dependency JARs — no persistence.xml configuration needed. Established in casehub-ledger#189 (T1.1-T1.2). Verified cross-JAR with 3-JAR chain (api → runtime → consumer).

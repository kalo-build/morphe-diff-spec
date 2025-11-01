# Morphe Diff Specification (KA:MD1) - Semantic Schema Evolution

## Table of Contents

- [Morphe Diff - Semantic Schema Evolution Specification](#morphe-diff---semantic-schema-evolution-specification)
  - [Overview](#overview)
  - [Specification Versions](#specification-versions)
  - [Specification Hierarchy](#specification-hierarchy)
  - [Key Features](#key-features)
  - [Motivation](#motivation)
  - [Delta Operations](#delta-operations)
    - [Add](#add)
    - [Remove](#remove)
    - [Modify](#modify)
    - [Rename](#rename)
    - [Move](#move)
    - [Deprecate](#deprecate)
  - [Change Classification](#change-classification)
    - [Breaking Changes](#breaking-changes)
    - [Additive Changes](#additive-changes)
    - [Safe Changes](#safe-changes)
  - [Artifact Types](#artifact-types)
    - [Model Deltas](#model-deltas)
    - [Entity Deltas](#entity-deltas)
    - [Structure Deltas](#structure-deltas)
    - [Enum Deltas](#enum-deltas)
    - [Field Deltas](#field-deltas)
    - [Relationship Deltas](#relationship-deltas)
  - [Diff Metadata](#diff-metadata)
  - [Use Cases](#use-cases)
  - [Plugin Integration](#plugin-integration)
  - [License](#license)

## Overview

Morphe Diff (KA:MD1) is a semantic schema evolution specification that describes changes between two versions of a Morphe schema. It produces deterministic, human-readable, and machine-consumable delta artifacts that power SQL migrations, API documentation updates, type regeneration, and other downstream code generation tasks.

Unlike low-level SQL diffs that focus on DDL statements, Morphe Diff operates at the semantic layer, capturing the *intent* and *meaning* of schema changes independent of any specific target format.

## Specification Versions

Version | Status | Description | Docs
--------|---------|------------|------
KA:MD1 | 🚧 In Progress | First stable release of the Morphe Diff specification | This document
KA:MD1:YAML1 | 🚧 In Progress | YAML format standard for KA:MD1 | [format/YAML.md](format/YAML.md)

## Specification Hierarchy

The Morphe Diff specification follows the same hierarchical naming scheme as Morphe:

### Base Specification (`KA:MD1`)
- `KA:` - Organization/context prefix
- `MD1` - Base specification identifier (Morphe Diff version 1)
- *Example:* `KA:MD1` - The core Morphe Diff specification

### Format Specification (`KA:MD1:YAML1`)
- Extends base specification with format identifier
- `YAML1` - Format representation identifier (YAML version 1)
- *Example:* `KA:MD1:YAML1` - The YAML format specification for Morphe Diff

### Base Format
Like Morphe, **YAML** (`KA:MD1:YAML1`) serves as the base format for Morphe Diff, providing:

- **Canonical Reference**: All delta artifacts use YAML as the primary format
- **Human Readability**: Easy to review in pull requests and CI/CD pipelines
- **Machine Consumable**: Straightforward parsing for migration generators and validators
- **Version Control Friendly**: Clean diffs in Git and other VCS systems

### Relationship to Morphe Specification
Morphe Diff is a companion specification to Morphe (`KA:MO1`):

- **Morphe (`KA:MO1`)**: Defines data models and structures
- **Morphe Diff (`KA:MD1`)**: Defines changes *between* versions of Morphe schemas

## Key Features

1. 🔍 **Semantic Change Detection**
   * Detect additions, removals, modifications, renames, and moves
   * Identify breaking vs. additive vs. safe changes
   * Preserve intent across format transpilation
   * Track structural fingerprints for rename detection

2. 🎯 **Deterministic Deltas**
   * Consistent output for identical schema changes
   * Canonical normalization before diffing
   * Stable ordering of operations
   * Reproducible across environments

3. 🔄 **Multi-Purpose Output**
   * Drive PostgreSQL migration scripts
   * Update OpenAPI specifications
   * Regenerate TypeScript types
   * Validate API contracts
   * Power form field updates

4. 🛡️ **Safety & Visibility**
   * Clear breaking change warnings
   * Deprecation tracking
   * CI/CD integration ready
   * Human-readable review format

## Motivation

Schema evolution is a critical and risky aspect of application development. Traditional approaches have several pain points:

**Manual SQL Migration Challenges:**
- Error-prone hand-written DDL statements
- Inconsistent naming and formatting
- No semantic understanding of changes
- Difficult to review in pull requests
- Hard to coordinate across teams

**Code Generation Synchronization:**
When a Morphe schema changes, multiple code artifacts need updating:
- Database migration scripts
- Backend type definitions
- API documentation
- Frontend types
- Form validators
- Test fixtures

**Existing Tool Limitations:**
- SQL schema diff tools focus on DDL syntax, not semantic meaning
- ORM migration generators are language/framework-specific
- No single source of truth for cross-language evolution

**Morphe Diff Solution:**
By capturing schema changes at the semantic Morphe layer:
1. Generate accurate, intent-preserving deltas
2. Power multiple downstream code generators from a single diff
3. Enable safe, reviewable schema evolution
4. Integrate seamlessly into CI/CD pipelines
5. Detect subtle issues like inadvertent breaking changes

## Delta Operations

Morphe Diff defines six fundamental delta operations that describe all possible schema changes.

### Add

Indicates a new artifact was introduced to the schema.

**Applies to:** Models, Entities, Structures, Enums, Fields, Relationships

**Example:**
```yaml
operation: add
type: field
target:
  model: User
  field: PhoneNumber
definition:
  type: String
  attributes:
    - nullable
```

**Downstream Impact:**
- SQL: `ALTER TABLE` with `ADD COLUMN`
- TypeScript: New optional property
- OpenAPI: New schema property

### Remove

Indicates an artifact was deleted from the schema.

**Applies to:** Models, Entities, Structures, Enums, Fields, Relationships

**Example:**
```yaml
operation: remove
type: field
target:
  model: User
  field: LegacyID
classification: breaking
```

**Downstream Impact:**
- SQL: `ALTER TABLE` with `DROP COLUMN`
- TypeScript: Property removal (breaking change)
- OpenAPI: Property removal from schema

### Modify

Indicates an artifact's properties changed without changing its identity.

**Applies to:** Fields (type changes, attribute changes), Relationships (cardinality changes), Enums (entry changes)

**Example:**
```yaml
operation: modify
type: field
target:
  model: User
  field: Email
changes:
  attributes:
    before:
      - nullable
    after:
      - mandatory
classification: breaking
```

**Downstream Impact:**
- SQL: Constraint modifications, type alterations
- TypeScript: Type changes (nullable → required)
- OpenAPI: Schema constraint updates

### Rename

Indicates an artifact's name changed while preserving its identity and semantics.

**Applies to:** Models, Entities, Structures, Enums, Fields, Relationships

**Example:**
```yaml
operation: rename
type: field
target:
  model: User
  field: PhoneNr
renamed_to: PhoneNumber
fingerprint: "field:User.contact_phone:String:nullable"
classification: safe
```

**Downstream Impact:**
- SQL: `ALTER TABLE` with `RENAME COLUMN`
- TypeScript: Alias or gradual migration
- OpenAPI: Property name change with backward compatibility

### Move

Indicates an artifact relocated within the schema structure (e.g., field moved from one model to another via refactoring).

**Applies to:** Fields (model-to-model), Enum entries (enum-to-enum)

**Example:**
```yaml
operation: move
type: field
source:
  model: User
  field: BillingAddress
destination:
  model: Account
  field: BillingAddress
classification: breaking
```

**Downstream Impact:**
- SQL: Complex data migration (cross-table)
- TypeScript: Structural refactoring
- OpenAPI: Schema reorganization

### Deprecate

Indicates an artifact is marked for future removal but still present.

**Applies to:** Models, Entities, Structures, Enums, Fields, Relationships

**Example:**
```yaml
operation: deprecate
type: field
target:
  model: User
  field: LegacyAPIKey
deprecation:
  since: "2025-01-15"
  remove_after: "2025-07-15"
  migration_guide: "Use APIKeyV2 field instead"
classification: additive
```

**Downstream Impact:**
- SQL: Comment additions
- TypeScript: `@deprecated` annotations
- OpenAPI: `deprecated: true` property

## Change Classification

Every delta operation is classified by its impact on existing consumers.

### Breaking Changes

Changes that **require** consumer updates or will cause runtime failures.

**Examples:**
- Remove a field, model, or relationship
- Modify field type in an incompatible way (e.g., String → Integer)
- Change field from nullable to mandatory
- Remove enum entries
- Modify relationship cardinality (One → Many)

**Marker:** `classification: breaking`

**CI/CD Impact:** Should trigger warnings, require approval, or block merging

### Additive Changes

Changes that add new capabilities without affecting existing consumers.

**Examples:**
- Add a new field with nullable or default value
- Add a new model, entity, structure
- Add new enum entries
- Add new relationships
- Deprecate (without removing) an artifact

**Marker:** `classification: additive`

**CI/CD Impact:** Generally safe to auto-approve

### Safe Changes

Changes that preserve backward compatibility and don't affect runtime behavior.

**Examples:**
- Rename with proper migration support
- Reorder fields (when order is not semantically meaningful)
- Add documentation or comments
- Add non-enforced attributes

**Marker:** `classification: safe`

**CI/CD Impact:** Safe to auto-approve

## Artifact Types

### Model Deltas

Changes to persisted data structures (`.mod` files).

**Supported Operations:** Add, Remove, Rename, Deprecate

**Example:**
```yaml
operation: add
type: model
target:
  model: Organization
definition:
  fields:
    ID:
      type: UUID
      attributes:
        - mandatory
    Name:
      type: String
  identifiers:
    primary: ID
classification: additive
```

### Entity Deltas

Changes to business data aggregation structures (`.ent` files).

**Supported Operations:** Add, Remove, Rename, Modify, Deprecate

**Example:**
```yaml
operation: modify
type: entity
target:
  entity: UserProfile
changes:
  fields:
    added:
      - name: CompanyName
        type: User.Company.Name
    removed:
      - name: LegacyCompanyID
classification: breaking
```

### Structure Deltas

Changes to non-persisted field groupings (`.str` files).

**Supported Operations:** Add, Remove, Rename, Modify, Deprecate

**Example:**
```yaml
operation: rename
type: structure
target:
  structure: PostalAddress
renamed_to: MailingAddress
fingerprint: "structure:address_fields:4_fields"
classification: safe
```

### Enum Deltas

Changes to constant value sets (`.enum` files).

**Supported Operations:** Add, Remove, Rename, Modify (entries)

**Example:**
```yaml
operation: modify
type: enum
target:
  enum: UserRole
changes:
  entries:
    added:
      Moderator: MOD
    removed:
      LegacyAdmin: LEGACY_ADMIN
classification: breaking
```

### Field Deltas

Changes to individual fields within models, entities, or structures.

**Supported Operations:** Add, Remove, Modify, Rename, Move

**Example:**
```yaml
operation: modify
type: field
target:
  model: User
  field: CreatedAt
changes:
  type:
    before: String
    after: Time
  attributes:
    before: []
    after:
      - immutable
classification: breaking
```

### Relationship Deltas

Changes to relationships between models or entities.

**Supported Operations:** Add, Remove, Modify, Rename

**Example:**
```yaml
operation: add
type: relationship
target:
  model: User
  relationship: ProfileImage
definition:
  type: HasOne
classification: additive
```

## Diff Metadata

Every Morphe Diff artifact includes metadata about the diff context:

```yaml
metadata:
  spec_version: KA:MD1:YAML1
  source:
    commit: abc123def
    branch: main
    timestamp: "2025-01-15T10:30:00Z"
  target:
    commit: def456ghi
    branch: feature/add-phone-field
    timestamp: "2025-01-20T14:45:00Z"
  summary:
    total_changes: 12
    breaking: 2
    additive: 8
    safe: 2
  generated_at: "2025-01-20T15:00:00Z"
  generator: kalo-morphe-diff@1.0.0
```

## Use Cases

Morphe Diff artifacts enable various downstream code generation and validation workflows:

### 1. SQL Migration Generation
Diff artifacts describing schema changes can be consumed by SQL migration plugins to generate database migration scripts (e.g., PostgreSQL `ALTER TABLE` statements).

### 2. Type System Updates
Plugins can use diff artifacts to incrementally update type definitions in TypeScript, Go, Python, or other languages without full regeneration.

### 3. API Contract Validation
Breaking change detection plugins can analyze diff artifacts to enforce API versioning policies and backward compatibility requirements.

### 4. Documentation Generation
Changelog and API documentation plugins can consume diff artifacts to automatically generate human-readable migration guides.

### 5. Schema Evolution Safety
Analysis plugins can inspect diff metadata to provide warnings, block deployments, or require manual review based on change classification.

## Plugin Integration

Morphe Diff artifacts are designed to serve as input to compile plugins within build systems. Plugins receive structured delta information that describes semantic changes between schema versions, enabling them to generate appropriate migration code, type updates, or validation reports.

The diff artifact format is independent of how it is generated or consumed, allowing flexibility in tooling implementations while maintaining a consistent semantic representation of schema evolution.

## License

This project is licensed under the MIT License.


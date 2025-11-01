# YAML Format Specification (`KA:MD1:YAML1`)

## Overview

This document specifies the YAML format for Morphe Diff (`KA:MD1`) delta artifacts. The `KA:MD1:YAML1` format serves as the canonical base format for describing semantic schema changes between two Morphe schema versions.

## Supported Features

The `KA:MD1:YAML1` format supports all core Morphe Diff operations:

✅ **Add** - New artifacts introduced to the schema  
✅ **Remove** - Artifacts deleted from the schema  
✅ **Modify** - Changes to artifact properties  
✅ **Rename** - Identity-preserving name changes  
✅ **Move** - Artifact relocation within schema structure  
✅ **Deprecate** - Artifacts marked for future removal

## File Extensions

- `.diff.yaml` - Morphe Diff delta artifacts
- `.diff.yml` - Alternative extension

## Document Structure

Every Morphe Diff YAML document consists of two main sections:

1. **Metadata** - Context about the diff (source/target versions, summary, etc.)
2. **Changes** - List of delta operations

### Basic Structure

```yaml
metadata:
  spec_version: KA:MD1:YAML1
  source:
    commit: abc123def
    branch: main
    timestamp: "2025-01-15T10:30:00Z"
  target:
    commit: def456ghi
    branch: feature/add-user-phone
    timestamp: "2025-01-20T14:45:00Z"
  summary:
    total_changes: 5
    breaking: 1
    additive: 3
    safe: 1
  generated_at: "2025-01-20T15:00:00Z"
  generator: kalo-morphe-diff@1.0.0

changes:
  - operation: add
    type: field
    target:
      model: User
      field: PhoneNumber
    definition:
      type: String
      attributes:
        - nullable
    classification: additive
```

## Delta Operations

### Add Operations

#### Add Model

```yaml
- operation: add
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
      TaxID:
        type: String
    identifiers:
      primary: ID
    related:
      Owner:
        type: ForOne
  classification: additive
```

#### Add Field

```yaml
- operation: add
  type: field
  target:
    model: User
    field: PhoneNumber
  definition:
    type: String
    attributes:
      - nullable
  classification: additive
```

#### Add Relationship

```yaml
- operation: add
  type: relationship
  target:
    model: User
    relationship: ProfileImage
  definition:
    type: HasOne
  classification: additive
```

#### Add Enum

```yaml
- operation: add
  type: enum
  target:
    enum: UserStatus
  definition:
    type: String
    entries:
      Active: ACTIVE
      Suspended: SUSPENDED
      Deleted: DELETED
  classification: additive
```

#### Add Entity

```yaml
- operation: add
  type: entity
  target:
    entity: UserProfile
  definition:
    fields:
      ID:
        type: User.ID
        attributes:
          - immutable
          - mandatory
      FullName:
        type: User.FullName
      Email:
        type: User.ContactInfo.Email
    identifiers:
      primary: ID
  classification: additive
```

#### Add Structure

```yaml
- operation: add
  type: structure
  target:
    structure: GeoCoordinates
  definition:
    fields:
      Latitude:
        type: Float
      Longitude:
        type: Float
      Altitude:
        type: Float
  classification: additive
```

### Remove Operations

#### Remove Model

```yaml
- operation: remove
  type: model
  target:
    model: LegacyUser
  reason: "Consolidated into User model"
  classification: breaking
```

#### Remove Field

```yaml
- operation: remove
  type: field
  target:
    model: User
    field: LegacyID
  reason: "Replaced by UUID-based ID"
  classification: breaking
```

#### Remove Relationship

```yaml
- operation: remove
  type: relationship
  target:
    model: User
    relationship: LegacyProfile
  classification: breaking
```

#### Remove Enum

```yaml
- operation: remove
  type: enum
  target:
    enum: OldUserRole
  reason: "Replaced by UserRole enum"
  classification: breaking
```

#### Remove Enum Entry

```yaml
- operation: modify
  type: enum
  target:
    enum: UserRole
  changes:
    entries:
      removed:
        - symbol: LegacyAdmin
          value: LEGACY_ADMIN
  reason: "Admin role restructuring"
  classification: breaking
```

### Modify Operations

#### Modify Field Type

```yaml
- operation: modify
  type: field
  target:
    model: User
    field: CreatedAt
  changes:
    type:
      before: String
      after: Time
  reason: "Improve type safety and timezone handling"
  classification: breaking
```

#### Modify Field Attributes

```yaml
- operation: modify
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
        - unique
  classification: breaking
```

#### Modify Relationship Cardinality

```yaml
- operation: modify
  type: relationship
  target:
    model: User
    relationship: Address
  changes:
    type:
      before: HasOne
      after: HasMany
  reason: "Support multiple addresses per user"
  classification: breaking
```

#### Modify Enum Entries

```yaml
- operation: modify
  type: enum
  target:
    enum: UserRole
  changes:
    entries:
      added:
        - symbol: Moderator
          value: MOD
        - symbol: Contributor
          value: CONTRIB
      removed:
        - symbol: LegacyAdmin
          value: LEGACY_ADMIN
      modified:
        - symbol: Admin
          before: ADMIN
          after: ADMINISTRATOR
  classification: breaking
```

#### Modify Polymorphic Relationship

```yaml
- operation: modify
  type: relationship
  target:
    model: Comment
    relationship: Commentable
  changes:
    for:
      before:
        - Person
        - Company
      after:
        - Person
        - Company
        - Post
  classification: breaking
```

### Rename Operations

#### Rename Model

```yaml
- operation: rename
  type: model
  target:
    model: Organisation
  renamed_to: Organization
  fingerprint: "model:organisation:6_fields:2_relationships"
  classification: safe
  migration:
    strategy: alias
    duration: "3 months"
```

#### Rename Field

```yaml
- operation: rename
  type: field
  target:
    model: User
    field: PhoneNr
  renamed_to: PhoneNumber
  fingerprint: "field:User.phone_contact:String:nullable"
  classification: safe
```

#### Rename Relationship

```yaml
- operation: rename
  type: relationship
  target:
    model: User
    relationship: Addr
  renamed_to: Address
  classification: safe
```

#### Rename Enum

```yaml
- operation: rename
  type: enum
  target:
    enum: UserType
  renamed_to: UserRole
  fingerprint: "enum:user_classification:String:5_entries"
  classification: safe
```

#### Rename Entity

```yaml
- operation: rename
  type: entity
  target:
    entity: PersonView
  renamed_to: PersonProfile
  classification: safe
```

### Move Operations

#### Move Field Between Models

```yaml
- operation: move
  type: field
  source:
    model: User
    field: BillingAddress
  destination:
    model: Account
    field: BillingAddress
  reason: "Billing moved to account-level management"
  classification: breaking
  migration:
    data_migration_required: true
    rollback_supported: true
```

#### Move Enum Entry

```yaml
- operation: move
  type: enum_entry
  source:
    enum: GlobalStatus
    entry: Pending
  destination:
    enum: UserStatus
    entry: Pending
  classification: breaking
```

### Deprecate Operations

#### Deprecate Model

```yaml
- operation: deprecate
  type: model
  target:
    model: LegacyUser
  deprecation:
    since: "2025-01-15"
    remove_after: "2025-07-15"
    migration_guide: "Migrate to User model using the consolidation script"
    replacement: User
  classification: additive
```

#### Deprecate Field

```yaml
- operation: deprecate
  type: field
  target:
    model: User
    field: LegacyAPIKey
  deprecation:
    since: "2025-01-15"
    remove_after: "2025-07-15"
    migration_guide: "Use APIKeyV2 field instead"
    replacement: APIKeyV2
  classification: additive
```

#### Deprecate Relationship

```yaml
- operation: deprecate
  type: relationship
  target:
    model: User
    relationship: LegacyProfile
  deprecation:
    since: "2025-01-15"
    remove_after: "2025-07-15"
    migration_guide: "Use UserProfile entity instead"
  classification: additive
```

#### Deprecate Enum

```yaml
- operation: deprecate
  type: enum
  target:
    enum: OldUserRole
  deprecation:
    since: "2025-01-15"
    remove_after: "2025-07-15"
    migration_guide: "Replace with UserRole enum"
    replacement: UserRole
  classification: additive
```

## Change Classification

Every delta operation must include a `classification` field:

```yaml
classification: breaking  # Requires consumer updates
classification: additive  # Adds capabilities without breaking
classification: safe      # Backward compatible, non-breaking
```

### Breaking Changes

Changes that require consumer updates:

```yaml
- operation: remove
  type: field
  target:
    model: User
    field: LegacyID
  classification: breaking  # Consumers using this field will break
```

```yaml
- operation: modify
  type: field
  target:
    model: User
    field: Age
  changes:
    type:
      before: String
      after: Integer
  classification: breaking  # Type incompatibility
```

### Additive Changes

Changes that add new capabilities:

```yaml
- operation: add
  type: field
  target:
    model: User
    field: PhoneNumber
  definition:
    type: String
    attributes:
      - nullable
  classification: additive  # New optional field
```

### Safe Changes

Changes that preserve full backward compatibility:

```yaml
- operation: rename
  type: field
  target:
    model: User
    field: PhoneNr
  renamed_to: PhoneNumber
  classification: safe  # Can be aliased during migration
```

## Metadata Section

### Required Metadata Fields

```yaml
metadata:
  spec_version: KA:MD1:YAML1  # Required: Format version
  source:                      # Required: Base schema version
    commit: abc123def
    branch: main
    timestamp: "2025-01-15T10:30:00Z"
  target:                      # Required: Target schema version
    commit: def456ghi
    branch: feature/updates
    timestamp: "2025-01-20T14:45:00Z"
  generated_at: "2025-01-20T15:00:00Z"  # Required: When diff was generated
```

### Optional Metadata Fields

```yaml
metadata:
  # ... required fields ...
  summary:
    total_changes: 12
    breaking: 2
    additive: 8
    safe: 2
    by_type:
      model: 1
      field: 7
      relationship: 2
      enum: 2
  generator: kalo-morphe-diff@1.0.0
  options:
    rename_detection: true
    fingerprint_algorithm: structural_hash_v1
    ignore_attributes:
      - internal_comment
  notes: "Schema update for user phone number feature"
```

## Complete Example

```yaml
metadata:
  spec_version: KA:MD1:YAML1
  source:
    commit: abc123def456
    branch: main
    timestamp: "2025-01-15T10:30:00Z"
  target:
    commit: def456ghi789
    branch: feature/user-enhancements
    timestamp: "2025-01-20T14:45:00Z"
  summary:
    total_changes: 6
    breaking: 2
    additive: 3
    safe: 1
  generated_at: "2025-01-20T15:00:00Z"
  generator: kalo-morphe-diff@1.0.0

changes:
  # Add new field (additive)
  - operation: add
    type: field
    target:
      model: User
      field: PhoneNumber
    definition:
      type: String
      attributes:
        - nullable
    classification: additive

  # Add new enum (additive)
  - operation: add
    type: enum
    target:
      enum: PhoneType
    definition:
      type: String
      entries:
        Mobile: MOBILE
        Home: HOME
        Work: WORK
    classification: additive

  # Rename field (safe)
  - operation: rename
    type: field
    target:
      model: User
      field: PhoneNr
    renamed_to: PhoneNumber
    fingerprint: "field:User.phone:String:nullable"
    classification: safe

  # Remove deprecated field (breaking)
  - operation: remove
    type: field
    target:
      model: User
      field: LegacyContactNumber
    reason: "Replaced by PhoneNumber field"
    classification: breaking

  # Modify field constraint (breaking)
  - operation: modify
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
          - unique
    classification: breaking

  # Add new relationship (additive)
  - operation: add
    type: relationship
    target:
      model: User
      relationship: PhoneNumbers
    definition:
      type: HasMany
    classification: additive
```

## Fingerprinting for Rename Detection

Renames include a `fingerprint` field that helps identify structurally identical artifacts across versions:

```yaml
- operation: rename
  type: field
  target:
    model: User
    field: OldFieldName
  renamed_to: NewFieldName
  fingerprint: "field:User.semantic_id:String:nullable:indexed"
  classification: safe
```

### Fingerprint Format

Fingerprints are colon-delimited strings containing:
- Artifact type
- Semantic identifiers (not names)
- Structural properties
- Attribute signatures

**Examples:**
```
field:User.contact_primary:String:nullable
model:user_entity:8_fields:3_relationships
enum:role_classification:String:5_entries
relationship:user_to_company:ForOne:aliased
```

## Notes and Documentation

Operations can include optional `notes` and `reason` fields:

```yaml
- operation: modify
  type: field
  target:
    model: User
    field: Status
  changes:
    type:
      before: String
      after: UserStatus  # Enum reference
  reason: "Enforce valid status values"
  notes: |
    This change requires data migration to convert existing
    string values to enum entries. Migration script available
    in migrations/20250120_convert_user_status.sql
  classification: breaking
```

## Relationship to Morphe Specification

Morphe Diff artifacts reference Morphe specification constructs:

- **Field Types**: Must be valid Morphe field types (`String`, `Integer`, etc.) or enum references
- **Relationship Types**: Must be valid Morphe relationship types (`HasOne`, `ForMany`, etc.)
- **Identifiers**: Must reference existing or newly added identifier definitions
- **Attributes**: Free-form attributes as defined in Morphe

## Validation Rules

### Required Fields by Operation

**Add:**
- `operation: add`
- `type: <artifact_type>`
- `target: <location>`
- `definition: <complete_definition>`
- `classification: <classification>`

**Remove:**
- `operation: remove`
- `type: <artifact_type>`
- `target: <location>`
- `classification: breaking` (always breaking)

**Modify:**
- `operation: modify`
- `type: <artifact_type>`
- `target: <location>`
- `changes: <before_after_diff>`
- `classification: <classification>`

**Rename:**
- `operation: rename`
- `type: <artifact_type>`
- `target: <location>`
- `renamed_to: <new_name>`
- `classification: safe` (usually safe)

**Move:**
- `operation: move`
- `type: <artifact_type>`
- `source: <source_location>`
- `destination: <destination_location>`
- `classification: breaking` (usually breaking)

**Deprecate:**
- `operation: deprecate`
- `type: <artifact_type>`
- `target: <location>`
- `deprecation: <deprecation_details>`
- `classification: additive` (always additive)

## Target Location Formats

### Model Target
```yaml
target:
  model: User
```

### Field Target
```yaml
target:
  model: User
  field: Email
```

### Relationship Target
```yaml
target:
  model: User
  relationship: ContactInfo
```

### Entity Target
```yaml
target:
  entity: UserProfile
```

### Entity Field Target
```yaml
target:
  entity: UserProfile
  field: Email
```

### Enum Target
```yaml
target:
  enum: UserRole
```

### Structure Target
```yaml
target:
  structure: Address
```

## License

This project is licensed under the MIT License.


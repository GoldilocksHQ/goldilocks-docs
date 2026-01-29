# OpenAPI Documentation Sync with Backend Implementation

**Created**: 2026-01-29
**Last Updated**: 2026-01-29
**Author**: AI Assistant
**Status**: 🟢 COMPLETED
**Complexity**: MEDIUM (OpenAPI schema updates, no code changes)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Context and Motivation](#context-and-motivation)
3. [Discrepancy Analysis](#discrepancy-analysis)
4. [Goals and Requirements](#goals-and-requirements)
5. [Design Decisions](#design-decisions)
6. [Implementation Phases](#implementation-phases)
7. [Testing Strategy](#testing-strategy)
8. [Success Criteria](#success-criteria)
9. [Status Legend](#status-legend)

---

## Executive Summary

### Objective

Synchronize the `openapi.yaml` specification in `goldilocks-docs` with the actual backend implementation in `goldilocks-backend`. The documentation has fallen out of sync with several backend updates, resulting in 6 discrepancies that need to be corrected.

### Scope

This implementation plan covers 6 discrepancies between documentation and backend:

| # | Discrepancy | Severity | Phase |
|---|-------------|----------|-------|
| 1 | `profiles_requested` missing from QueryRequestV1 | **High** | 1 |
| 2 | `user_id`, `api_key_id` missing from SearchQueryResponse | Medium | 2 |
| 3 | SearchProfilesMetadata completely wrong | **High** | 3 |
| 4 | `user_id` missing from MoreProfilesJobResponse | Low | 2 |
| 5 | `data.metadata` in enrichment example doesn't exist | **High** | 4 |
| 6 | EnrichmentResponse schema has nested metadata that doesn't exist | Medium | 4 |

**Out of Scope**: Lists API and Conversations API documentation (to be documented separately).

### Key Decisions (Confirmed)

1. ✅ **SearchProfilesMetadata**: Use simplified backend structure (`total_profiles` only)
2. ✅ **User-facing fields**: Document `user_id` and `api_key_id` in responses
3. ✅ **Enrichment examples**: Adjust to match reality (remove non-existent `data.metadata`)

### Impact

- **Files Affected**: `openapi.yaml` only
- **User Impact**: Documentation will accurately reflect API behavior
- **Breaking Changes**: None (documentation-only changes)
- **Backward Compatibility**: Not required (explicit instruction)

---

## Context and Motivation

### Problem Statement

The `openapi.yaml` specification in the documentation site does not accurately reflect the backend implementation. This causes:

1. **Developer Confusion**: API users expecting documented fields that don't exist (or missing fields that do exist)
2. **Integration Issues**: Developers building clients based on incorrect schemas
3. **Support Overhead**: Questions about why responses don't match documentation
4. **Trust Issues**: Inaccurate documentation undermines confidence in the API

### Root Cause

The backend has evolved through multiple implementation phases (Phase 2 Response Envelope, Phase 3 Query Endpoint, Phase 4 Database Restructuring, Phase 6 More Profiles) without corresponding documentation updates.

### Business Value

- **Improved Developer Experience**: Accurate documentation reduces confusion
- **Reduced Support Load**: Fewer questions about API behavior
- **Faster Integration**: Developers can rely on documentation accuracy
- **Professional Appearance**: Up-to-date docs reflect well on the product

---

## Discrepancy Analysis

### Discrepancy 1: QueryRequestV1 Schema - Missing `profiles_requested` Field

**Severity**: High
**Phase**: 1

**Current Documentation** (`openapi.yaml` lines 115-125):
```yaml
QueryRequestV1:
  type: object
  required:
    - query
  properties:
    query:
      type: string
      minLength: 1
      description: Natural language search query
```

**Actual Backend Implementation** (`goldilocks_schemas/query.py` lines 49-77):
```python
class QueryRequestV1(BaseModel):
    query: str = Field(..., min_length=1, description="User's query text")
    profiles_requested: Optional[int] = Field(
        default=20,
        ge=1,
        le=200,
        description="Number of profiles requested (default: 20, min: 1, max: 200)"
    )
```

**Fix Required**: Add `profiles_requested` optional field with default 20, min 1, max 200.

---

### Discrepancy 2: SearchQueryResponse - Missing Fields

**Severity**: Medium
**Phase**: 2

**Current Documentation** (`openapi.yaml` lines 164-179):
```yaml
SearchQueryResponse:
  properties:
    job_id:
      type: string
      format: uuid
    query:
      type: string
```

**Actual Backend Implementation** (`searches.py` lines 529-535):
```python
response_data = {
    "job_id": job_id,
    "query": query_data.query,
    "user_id": str(context.user_id),
    "api_key_id": str(context.api_key_id) if context.api_key_id else None
}
```

**Fix Required**: Add `user_id` (required) and `api_key_id` (nullable) fields.

---

### Discrepancy 3: SearchProfilesMetadata - Wrong Structure

**Severity**: High
**Phase**: 3

**Current Documentation** (`openapi.yaml` lines 446-473):
```yaml
SearchProfilesMetadata:
  properties:
    results:
      properties:
        total_possible_profiles_found:
          type: integer
        total_profiles_scanned:
          type: integer
        total_relevant_profiles_found:
          type: integer
```

**Actual Backend Implementation** (`metadata.py` lines 16-30):
```python
class SearchProfilesResultsMetadata(BaseModel):
    total_profiles: int = Field(
        ...,
        ge=0,
        description="Total number of profiles available for this search. "
                   "Calculated as min(relevant_profile_count, requested_profiles_count)."
    )

class SearchProfilesMetadata(BaseModel):
    results: SearchProfilesResultsMetadata
```

**Fix Required**: Replace entire schema with single `total_profiles` field.

---

### Discrepancy 4: MoreProfilesJobResponse - Missing `user_id` Field

**Severity**: Low
**Phase**: 2

**Current Documentation** (`openapi.yaml` lines 344-376):
```yaml
MoreProfilesJobResponse:
  required:
    - job_id
    - search_id
    - created_at
    - status
    - profiles_requested
  properties:
    job_id: ...
    search_id: ...
    created_at: ...
    status: ...
    profiles_requested: ...
```

**Actual Backend Implementation** (`search.py` lines 391-422):
```python
class MoreProfilesJobResponse(BaseModel):
    job_id: str
    search_id: uuid.UUID
    created_at: datetime
    user_id: str  # <-- Missing from docs
    status: str
    profiles_requested: int
```

**Fix Required**: Add `user_id` field to schema and required array.

---

### Discrepancy 5: Enrichment Response Example - Non-existent `data.metadata`

**Severity**: High
**Phase**: 4

**Current Documentation** (`openapi.yaml` lines 1390-1425) shows:
```yaml
example:
  success: true
  data:
    profiles: [...]
    metadata:           # <-- THIS DOES NOT EXIST
      total_profiles: 2
      profiles_processed: 2
      ...
  metadata:             # <-- This is correct (envelope level)
    results:
      total_profiles_requested_enrichment: 2
      ...
```

**Actual Backend Implementation** (`enrichment.py` lines 162-166):
```python
return ResponseBuilder.success(
    data={"profiles": result.profiles},  # Only profiles, NO metadata
    metadata=metadata.model_dump(),       # Metadata at envelope level
    start_time=start_time
)
```

**Fix Required**: Remove `data.metadata` from example entirely. Keep only envelope-level `metadata`.

---

### Discrepancy 6: EnrichmentResponse Schema - Has Nested Metadata That Doesn't Exist

**Severity**: Medium
**Phase**: 4

**Current Documentation** (`openapi.yaml` lines 269-342):
```yaml
EnrichmentResponse:
  type: object
  required:
    - profiles
    - metadata      # <-- WRONG: metadata is NOT inside data
  properties:
    profiles: ...
    metadata:       # <-- WRONG: This entire nested metadata structure doesn't exist
      type: object
      required:
        - total_profiles
        - profiles_processed
        ...
```

**Actual Backend Response Structure**:
```json
{
  "success": true,
  "data": {
    "profiles": [...]    // Only profiles, no metadata
  },
  "metadata": {          // Metadata is at envelope level
    "results": { ... }
  }
}
```

**Fix Required**: Remove `metadata` property from EnrichmentResponse schema entirely. EnrichmentResponse should only contain `profiles`.

---

## Goals and Requirements

### Functional Requirements

1. **QueryRequestV1**: Add `profiles_requested` optional field with:
   - Type: integer
   - Default: 20
   - Minimum: 1
   - Maximum: 200

2. **SearchQueryResponse**: Add fields:
   - `user_id` (string, UUID format, required)
   - `api_key_id` (string or null, UUID format)

3. **SearchProfilesMetadata**: Replace entire schema with:
   - `results.total_profiles` (integer, ≥0)
   - Remove all three old fields

4. **MoreProfilesJobResponse**: Add `user_id` field (string, required)

5. **EnrichmentResponse Schema**:
   - Remove `metadata` from required array
   - Remove entire `metadata` property (lines 280-341)
   - Keep only `profiles` property

6. **Enrichment Examples**: Remove `data.metadata` block from all examples

### Non-Functional Requirements

1. **Accuracy**: All schema definitions must match backend Pydantic models
2. **Completeness**: All response examples must be valid against their schemas
3. **Consistency**: Response format must be consistent across all endpoints
4. **Testability**: Changes can be validated by comparing with actual API responses

---

## Design Decisions

### Decision 1: SearchProfilesMetadata Simplification

**Decision**: Use the actual backend schema with single `total_profiles` field.

**Rationale**:
- The documented fields (`total_possible_profiles_found`, etc.) never existed in production
- The backend intentionally simplified this to a single meaningful metric
- `total_profiles` = min(relevant_profile_count, requested_profiles_count)
- Simpler is better for developer understanding

### Decision 2: Document All Response Fields

**Decision**: Include `user_id` and `api_key_id` in SearchQueryResponse.

**Rationale**:
- Complete documentation helps developers know exactly what to expect
- These fields are useful for debugging and correlation
- No surprises when integrating with the API

### Decision 3: Clean Separation of Envelope vs Data

**Decision**: EnrichmentResponse contains ONLY `profiles`. Metadata is at envelope level only.

**Rationale**:
- This is the consistent pattern across ALL endpoints
- The `APIResponse` envelope has `data`, `metadata`, `error` at root level
- No endpoint nests metadata inside data
- Matches backend ResponseBuilder.success() pattern

### Decision 4: No Backward Compatibility

**Decision**: Update documentation to match reality without worrying about old doc versions.

**Rationale**:
- User explicitly stated backward compatibility is not required
- Accurate documentation is more valuable than preserving incorrect schemas

---

## Implementation Phases

### Phase 1: Request Schemas (QueryRequestV1)

**Objective**: Fix the QueryRequestV1 schema to include `profiles_requested`.

**Status**: 🟢 COMPLETED

#### Task 1.1: Update QueryRequestV1 Schema Definition

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 115-125 (`components/schemas/QueryRequestV1`)
- **Changes**:
  ```yaml
  QueryRequestV1:
    type: object
    required:
      - query
    properties:
      query:
        type: string
        minLength: 1
        description: Natural language search query
        example: "Find partnership managers at Google in UK"
      profiles_requested:
        type: integer
        minimum: 1
        maximum: 200
        default: 20
        description: Number of profiles to return (1-200, default 20)
        example: 20
    description: Request body for executing a search query
  ```
- **Verification**: Schema includes `profiles_requested` with correct constraints

#### Task 1.2: Update Query Endpoint Request Examples

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 989-1001 (`paths/searches/query/requestBody/examples`)
- **Changes**:
  - "Basic search" example: Add `profiles_requested: 20`
  - "Specific requirements" example: Add `profiles_requested: 50`
  - "Search by linkedin url" example: Add `profiles_requested: 1`
- **Verification**: All examples include `profiles_requested` field

---

### Phase 2: Response Schemas (SearchQueryResponse, MoreProfilesJobResponse)

**Objective**: Add missing fields to response schemas.

**Status**: 🟢 COMPLETED

#### Task 2.1: Update SearchQueryResponse Schema

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 164-179 (`components/schemas/SearchQueryResponse`)
- **Changes**:
  ```yaml
  SearchQueryResponse:
    type: object
    required:
      - job_id
      - query
      - user_id
    properties:
      job_id:
        type: string
        format: uuid
        description: Job ID for tracking search progress via SSE
        example: "550e8400-e29b-41d4-a716-446655440000"
      query:
        type: string
        description: The original query text
        example: "Find partnership managers at Google in UK"
      user_id:
        type: string
        format: uuid
        description: User ID who initiated the search
        example: "123e4567-e89b-12d3-a456-426614174000"
      api_key_id:
        type: ["string", "null"]
        format: uuid
        description: API key ID used for the request (null if not applicable)
        example: "abc12345-e89b-12d3-a456-426614174000"
    description: Response data for POST /searches/query
  ```
- **Verification**: Schema matches backend response structure

#### Task 2.2: Update MoreProfilesJobResponse Schema

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 344-376 (`components/schemas/MoreProfilesJobResponse`)
- **Changes**:
  - Add `user_id` to required array
  - Add `user_id` property after `created_at`:
    ```yaml
    user_id:
      type: string
      format: uuid
      description: User ID who initiated the request
      example: "123e4567-e89b-12d3-a456-426614174000"
    ```
- **Verification**: Schema matches `goldilocks_schemas/search.py:MoreProfilesJobResponse`

#### Task 2.3: Update Query Endpoint Response Example

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 1034-1042 (`paths/searches/query/responses/201/example`)
- **Changes**:
  ```yaml
  example:
    success: true
    data:
      job_id: "550e8400-e29b-41d4-a716-446655440000"
      query: "Find partnership managers at Google in UK"
      user_id: "123e4567-e89b-12d3-a456-426614174000"
      api_key_id: "abc12345-e89b-12d3-a456-426614174000"
    metadata: null
    error: null
    request_id: "abc12345-e89b-12d3-a456-426614174000"
    processing_time_ms: 85
  ```
- **Verification**: Example matches schema

#### Task 2.4: Update More-Profiles Endpoint Response Example

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 1273-1284 (`paths/searches/{search_id}/more-profiles/responses/200/example`)
- **Changes**: Add `user_id: "123e4567-e89b-12d3-a456-426614174000"` to data object
- **Verification**: Example matches schema

---

### Phase 3: Metadata Schemas (SearchProfilesMetadata)

**Objective**: Replace incorrect SearchProfilesMetadata with correct structure.

**Status**: 🟢 COMPLETED

#### Task 3.1: Replace SearchProfilesMetadata Schema

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 446-473 (`components/schemas/SearchProfilesMetadata`)
- **Changes**: Replace entire schema with:
  ```yaml
  SearchProfilesMetadata:
    type: object
    required:
      - results
    properties:
      results:
        type: object
        required:
          - total_profiles
        properties:
          total_profiles:
            type: integer
            minimum: 0
            description: |
              Total number of profiles available for this search.
              Calculated as min(relevant_profile_count, requested_profiles_count).
              Represents the actual number of profiles the user can access.
            example: 20
    description: Metadata for GET /searches/{search_id}/profiles
  ```
- **Verification**: Schema matches `metadata.py:SearchProfilesMetadata`

#### Task 3.2: Update Search Profiles Endpoint Response Example

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 1149-1169 (`paths/searches/{search_id}/profiles/responses/200/example`)
- **Changes**: Update metadata section:
  ```yaml
  metadata:
    results:
      total_profiles: 20
  ```
- **Verification**: Example uses correct metadata structure

---

### Phase 4: Enrichment Schema Fixes

**Objective**: Fix EnrichmentResponse schema and remove non-existent `data.metadata`.

**Status**: 🟢 COMPLETED

#### Task 4.1: Update EnrichmentResponse Schema

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 269-342 (`components/schemas/EnrichmentResponse`)
- **Changes**: Replace entire schema with:
  ```yaml
  EnrichmentResponse:
    type: object
    required:
      - profiles
    properties:
      profiles:
        type: array
        items:
          $ref: '#/components/schemas/EnrichmentProfileData'
        description: List of enriched profiles with requested enrichment data
    description: |
      Response data for POST /enrichment/profiles.

      Note: This is the content of the 'data' field in the API envelope.
      Metadata (total_profiles_requested_enrichment, total_profiles_enriched, warnings)
      is provided at the envelope level via EnrichmentMetadataAPI, not inside this object.
  ```
- **Important**: Remove the entire `metadata` property (lines 280-341) and remove `metadata` from required array
- **Verification**: Schema shows `profiles` only, no nested `metadata`

#### Task 4.2: Verify EnrichmentMetadataAPI Schema

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 475-502 (`components/schemas/EnrichmentMetadataAPI`)
- **Action**: Verify existing schema matches backend (should already be correct)
- **Expected Schema**:
  ```yaml
  EnrichmentMetadataAPI:
    type: object
    required:
      - results
    properties:
      results:
        type: object
        required:
          - total_profiles_requested_enrichment
          - total_profiles_enriched
        properties:
          total_profiles_requested_enrichment:
            type: integer
            minimum: 0
            description: Total profiles requested for enrichment
          total_profiles_enriched:
            type: integer
            minimum: 0
            description: Total profiles successfully enriched
          warnings:
            type: array
            items:
              type: object
            description: Warnings about unavailable enrichment types
            default: []
    description: Metadata for POST /enrichment/profiles
  ```
- **Verification**: Schema matches `metadata.py:EnrichmentMetadata`

#### Task 4.3: Fix Enrichment Endpoint Response Example

- **Status**: 🟢 COMPLETED
- **File**: `openapi.yaml`
- **Location**: Lines 1390-1425 (`paths/enrichment/profiles/responses/200/example`)
- **Changes**: Remove `data.metadata` block entirely (lines 1398-1413). Final example:
  ```yaml
  example:
    success: true
    data:
      profiles:
        - people_id: "123e4567-e89b-12d3-a456-426614174000"
          email:
            - email: "john.doe@example.com"
              priority: 1
    metadata:
      results:
        total_profiles_requested_enrichment: 2
        total_profiles_enriched: 1
        warnings:
          - people_id: "123e4567-e89b-12d3-a456-426614174001"
            type: "W0001_ENRICHMENT_UNAVAILABLE_TYPE"
            enrichment_type: "email"
            reason: "Profile does not have email data"
    error: null
    request_id: "abc12345-e89b-12d3-a456-426614174000"
    processing_time_ms: 250
  ```
- **Verification**: Example shows correct structure without nested `data.metadata`

---

### Phase 5: Validation and Testing

**Objective**: Validate all changes and ensure consistency.

**Status**: 🟢 COMPLETED

#### Task 5.1: YAML Syntax Validation

- **Status**: 🟢 COMPLETED
- **Action**: Run YAML linter on `openapi.yaml`
- **Command**: `yamllint openapi.yaml` or IDE validation
- **Verification**: No syntax errors

#### Task 5.2: OpenAPI Specification Validation

- **Status**: 🟢 COMPLETED
- **Action**: Validate OpenAPI spec using validator tool
- **Tools**:
  - https://editor.swagger.io/
  - `npx @redocly/cli lint openapi.yaml`
- **Verification**: Spec is valid OpenAPI 3.1

#### Task 5.3: Local Preview Testing

- **Status**: 🟢 COMPLETED
- **Action**: Run `mint dev` and verify API playground works
- **Verification**:
  - All endpoints render correctly
  - Request/response examples display properly
  - Interactive playground functions correctly

#### Task 5.4: Cross-Reference with Backend

- **Status**: 🟢 COMPLETED
- **Action**: Compare each updated schema with backend implementation
- **Checklist**:

  | OpenAPI Schema | Backend File | Status |
  |----------------|--------------|--------|
  | QueryRequestV1 | `goldilocks_schemas/query.py:QueryRequestV1` | 🟢 |
  | SearchQueryResponse | `api/routers/api/searches.py` lines 529-535 | 🟢 |
  | SearchProfilesMetadata | `goldilocks_schemas/metadata.py:SearchProfilesMetadata` | 🟢 |
  | MoreProfilesJobResponse | `goldilocks_schemas/search.py:MoreProfilesJobResponse` | 🟢 |
  | EnrichmentResponse | `api/routers/api/enrichment.py` lines 162-166 | 🟢 |
  | EnrichmentMetadataAPI | `goldilocks_schemas/metadata.py:EnrichmentMetadata` | 🟢 |

- **Verification**: All schemas match backend implementations exactly

---

## Testing Strategy

### Manual Testing Checklist

- [x] **YAML Validation**: Run linter, no syntax errors
- [x] **OpenAPI Validation**: Validate against OpenAPI 3.1 spec
- [x] **Mintlify Preview**: `mint dev` renders without errors
- [x] **API Playground**: Interactive playground works correctly
- [x] **All Examples Valid**: Examples compile against their schemas

### Schema Verification Checklist

- [x] QueryRequestV1 has `profiles_requested` (optional, default 20, min 1, max 200)
- [x] SearchQueryResponse has `user_id` (required) and `api_key_id` (nullable)
- [x] SearchProfilesMetadata has ONLY `results.total_profiles`
- [x] MoreProfilesJobResponse has `user_id` (required)
- [x] EnrichmentResponse has ONLY `profiles` (no nested metadata)
- [x] EnrichmentMetadataAPI unchanged (already correct)
- [x] All examples match their schemas
- [x] No duplicate metadata in enrichment examples

---

## Success Criteria

### Functional Criteria

- [x] All 6 discrepancies resolved
- [x] OpenAPI spec validates as valid OpenAPI 3.1
- [x] Mintlify renders documentation correctly
- [x] API playground works with updated schemas

### Quality Criteria

- [x] All schemas match backend Pydantic models exactly
- [x] All examples are accurate and valid
- [x] Documentation is clear and consistent
- [x] No TypeScript/Pydantic validation errors in generated clients

### Documentation Criteria

- [x] All field descriptions are accurate
- [x] All constraints (min, max, default) documented
- [x] Response examples show realistic data
- [x] Metadata structure clearly documented at envelope level

---

## Status Legend

| Status | Symbol | Description |
|--------|--------|-------------|
| **Not Started** | 🔴 NOT_STARTED | Task not yet begun |
| **In Progress** | 🟡 IN_PROGRESS | Task actively being worked on |
| **Completed** | 🟢 COMPLETED | Task finished and verified |
| **Blocked** | ⚠️ BLOCKED | Task blocked by dependency |
| **Review** | 🔵 REVIEW | Task ready for review |

---

## Files to Modify

| File | Changes |
|------|---------|
| `openapi.yaml` | All schema and example updates |

---

## Risk Assessment

### Low Risk Items
- Task 1.2: Update request examples (simple text changes)
- Task 2.3, 2.4: Update response examples (simple text changes)
- Task 3.2: Update metadata example (simple text change)
- Task 4.2: Verify EnrichmentMetadataAPI (read-only verification)
- Task 5.1, 5.2, 5.3: Validation tasks (no file changes)

### Medium Risk Items
- Task 1.1: Update QueryRequestV1 (add new field)
- Task 2.1: Update SearchQueryResponse (add new fields)
- Task 2.2: Update MoreProfilesJobResponse (add new field)
- Task 4.3: Fix enrichment example (remove block)

### High Risk Items
- Task 3.1: Replace SearchProfilesMetadata (remove 3 fields, add 1)
- Task 4.1: Update EnrichmentResponse (remove large metadata block)

### Mitigation
- Validate YAML syntax after each phase
- Test with `mint dev` after each phase
- Keep backup of original `openapi.yaml` before starting

---

## Appendix A: Backend Reference Files

| Backend File | Contains |
|--------------|----------|
| `goldilocks_schemas/query.py` | QueryRequestV1 definition (lines 49-77) |
| `goldilocks_schemas/search.py` | MoreProfilesJobResponse definition (lines 391-422) |
| `goldilocks_schemas/metadata.py` | SearchProfilesMetadata (lines 16-30), EnrichmentMetadata (lines 50-59) |
| `goldilocks_schemas/enrichment.py` | Enrichment request/response schemas |
| `goldilocks_schemas/responses.py` | APIResponse envelope |
| `api/routers/api/searches.py` | Search endpoint implementations (lines 529-541, 440-459) |
| `api/routers/api/enrichment.py` | Enrichment endpoint (lines 162-166) |
| `core/http/responses/builder.py` | ResponseBuilder utility |

---

## Appendix B: Change Summary by Line Number

**openapi.yaml changes by location:**

| Lines | Current | Change |
|-------|---------|--------|
| 115-125 | QueryRequestV1 (query only) | Add `profiles_requested` field |
| 164-179 | SearchQueryResponse (2 fields) | Add `user_id`, `api_key_id` |
| 269-342 | EnrichmentResponse (profiles + metadata) | Remove entire `metadata` property |
| 344-376 | MoreProfilesJobResponse (5 fields) | Add `user_id` field |
| 446-473 | SearchProfilesMetadata (3 fields) | Replace with `total_profiles` only |
| 989-1001 | Query request examples | Add `profiles_requested` to each |
| 1034-1042 | Query response example | Add `user_id`, `api_key_id` |
| 1149-1169 | Search profiles response example | Fix metadata structure |
| 1273-1284 | More-profiles response example | Add `user_id` |
| 1390-1425 | Enrichment response example | Remove `data.metadata` block |

---

## Appendix C: Additional Work Completed

Beyond the original 6 discrepancies, the following additional improvements were made:

### SSE Job Stream Documentation Enhancements

**File**: `openapi.yaml` (lines 1419-1475)

1. **Clarified `search_id` Progressive Updates**:
   - Updated endpoint description to explain that `search_id` is provided during "Aggregate Search Results" stage
   - Documented that users can call `GET /searches/{search_id}/profiles` to retrieve profiles progressively
   - Updated metadata field description to highlight Aggregate Search Results stage

2. **Added New Example**: "Aggregate results (new profiles available)"
   - Shows `search_id`, `new_relevant_profiles`, and `total_relevant_profiles` in metadata
   - Demonstrates how to detect when new profiles are available during search

### Documentation Site Navigation

**File**: `docs.json`

- Removed "Guides" tab from navigation
- Simplified to show only "API Reference" content (Getting Started + Endpoints)
- Removed unused global anchors pointing to Mintlify docs/blog

---

**End of Implementation Plan**

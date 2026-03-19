# Feature Specification: n8n Helper Shadows

**Feature Branch**: `001-n8n-helper-shadows`
**Created**: 2026-03-13
**Status**: Draft

## Overview

The n8n Helper Shadows library provides a TypeScript shadow implementation of approximately 30-40 helper methods from n8n's `IExecuteFunctions` interface. These shadows map each n8n helper method to its Sinter equivalent, allowing existing n8n node code to compile against the shadow library while a normalizer pass rewrites the calls to native Sinter APIs. The goal is zero-friction migration of n8n community nodes into the Sinter ecosystem.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Developer migrates an n8n node using shadow wrappers (Priority: P1)

As an n8n node developer migrating to Sinter, I want to change my import from `n8n-workflow` to `@sinter/n8n-helpers` and have my existing code compile without modification so that I can migrate incrementally rather than rewriting everything at once.

**Why this priority**: The shadow library exists solely to enable this migration path; if imports don't swap cleanly the library has no value.

**Independent Test**: Take an existing open-source n8n node (e.g., the Slack node's execute method), swap the import, and run `tsc --strict`.

**Acceptance Scenarios**:

```
Scenario 1: getNodeParameter shadow
  Given an n8n node that calls this.getNodeParameter("resource", 0)
  When compiled against the shadow library
  Then it compiles without errors and at runtime returns the parameter value from the Sinter step config

Scenario 2: helpers.request shadow
  Given an n8n node that calls this.helpers.request(options)
  When compiled against the shadow library
  Then it compiles without errors and at runtime delegates to the Sinter HTTP client

Scenario 3: helpers.requestWithAuthentication shadow
  Given an n8n node that calls this.helpers.requestWithAuthentication("oAuth2Api", options)
  When compiled against the shadow library
  Then it compiles without errors and at runtime uses Sinter's credential management to authenticate the request

Scenario 4: getInputData shadow
  Given an n8n node that calls this.getInputData()
  When compiled against the shadow library
  Then it compiles without errors and at runtime returns the Sinter step's input data as n8n-shaped INodeExecutionData[]

Scenario 5: getWorkflowStaticData shadow
  Given an n8n node that calls this.getWorkflowStaticData("global")
  When compiled against the shadow library
  Then it compiles without errors and at runtime reads/writes Sinter's workflow-level persistent state
```

---

### User Story 2 - Normalizer rewrites shadow calls to native Sinter (Priority: P2)

As a transpilation pipeline developer, I want the shadow library's method signatures to be stable and well-documented so that the normalizer AST pass can reliably pattern-match and rewrite every shadow call to its Sinter-native equivalent.

**Why this priority**: The normalizer depends on stable shadow signatures but is developed after the shadow library is functional.

**Independent Test**: Run the normalizer on a file using shadow imports and verify every shadow call is rewritten to a Sinter API call.

**Acceptance Scenarios**:

```
Scenario 1: Normalizer rewrites getNodeParameter
  Given code calling shadows.getNodeParameter("resource", 0)
  When the normalizer pass runs
  Then the output code calls context.config["resource"] (Sinter native)

Scenario 2: Normalizer rewrites helpers.request
  Given code calling shadows.helpers.request(options)
  When the normalizer pass runs
  Then the output code calls sinter.http.request(options) (Sinter native)
```

---

### Edge Cases

- **Unsupported n8n method**: If a node calls a method not in the shadow set (e.g., `this.helpers.getBinaryDataBuffer`), the TypeScript compiler must produce a clear error indicating the method is not available, not a cryptic "property does not exist" message.
- **Parameter type mismatch**: n8n's `getNodeParameter` is heavily overloaded (string, number, boolean, JSON returns). The shadow must support all common overloads or document which are unsupported.
- **Credential types**: `requestWithAuthentication` supports dozens of credential types in n8n. The shadow must accept any string credential type name and map it to Sinter's generic credential resolution.
- **Pagination helpers**: n8n provides `helpers.requestWithAuthenticationPaginated`. If not shadowed, this must be documented as a known gap with a migration guide.
- **Binary data**: n8n nodes frequently use binary data helpers (`getBinaryDataBuffer`, `setBinaryDataKey`). These must either be shadowed or clearly listed as out-of-scope with alternatives.
- **Execution metadata**: Methods like `getExecutionId()`, `getMode()`, `getTimezone()` must return sensible Sinter equivalents or fixed values.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Library MUST provide a TypeScript implementation of `IExecuteFunctions` interface with shadow methods that compile against n8n's published type definitions.
- **FR-002**: Library MUST shadow the following core parameter methods: `getNodeParameter()`, `getNodeParameter()` with fallback, `getCurrentNodeParameter()`, `getCurrentNodeParameters()`.
- **FR-003**: Library MUST shadow the following HTTP methods: `helpers.request()`, `helpers.requestWithAuthentication()`, `helpers.httpRequest()`, `helpers.httpRequestWithAuthentication()`.
- **FR-004**: Library MUST shadow the following data methods: `getInputData()`, `getOutputData()`, `getWorkflowStaticData()`, `getWorkflowDataProxy()`.
- **FR-005**: Library MUST shadow the following execution methods: `getExecutionId()`, `getMode()`, `getTimezone()`, `getRestApiUrl()`, `getInstanceBaseUrl()`.
- **FR-006**: Library MUST shadow the following credential methods: `getCredentials()`.
- **FR-007**: Library MUST shadow the following node metadata methods: `getNode()`, `getNodeVersion()`, `getNodeType()`.
- **FR-008**: Library MUST shadow the following workflow methods: `getWorkflow()`, `getWorkflowId()`.
- **FR-009**: Library MUST shadow the following utility methods: `helpers.assertBinaryData()`, `helpers.prepareBinaryData()`, `helpers.copyBinaryFile()`, `helpers.returnJsonArray()`, `continueOnFail()`, `evaluateExpression()`.
- **FR-010**: Each shadow method MUST delegate to its Sinter equivalent at runtime rather than reimplementing n8n logic.
- **FR-011**: Library MUST export TypeScript type definitions (`.d.ts`) that match n8n's published interfaces for all shadowed methods.
- **FR-012**: Library MUST document every shadowed method with a comment indicating the Sinter equivalent it delegates to.
- **FR-013**: Library MUST throw a descriptive `NotImplementedError` for any `IExecuteFunctions` method that is intentionally not shadowed, listing the method name and suggesting a Sinter alternative.

### Key Entities

| Entity | Description |
|---|---|
| `ShadowExecuteFunctions` | Main class implementing the `IExecuteFunctions` shadow |
| `ShadowHelpers` | Nested object providing `request`, `httpRequest`, and binary data helpers |
| `ParameterResolver` | Internal component mapping `getNodeParameter` calls to Sinter config lookups |
| `CredentialResolver` | Internal component mapping n8n credential type names to Sinter secrets |
| `StaticDataStore` | Internal component mapping `getWorkflowStaticData` to Sinter persistent state |

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 30 n8n `IExecuteFunctions` methods have shadow implementations in the library.
- **SC-002**: Three real-world open-source n8n nodes (e.g., Slack, GitHub, HTTP Request) compile against the shadow library with zero type errors.
- **SC-003**: Every shadow method has a corresponding integration test that verifies it delegates to the correct Sinter API.
- **SC-004**: The normalizer can rewrite 100% of shadow calls in the three test nodes to Sinter-native equivalents.
- **SC-005**: The library's `.d.ts` exports match n8n's `IExecuteFunctions` interface for all shadowed methods — verified by a type compatibility test.
- **SC-006**: Methods intentionally not shadowed are documented in a compatibility matrix with migration guidance.

---

## v0.3 Addendum: Method Scope Clarification

*Added 2026-03-16 to align with master spec v0.3 §9.4 Track B*

### Method Count

The v0.3 master spec references **15–20 critical methods** as the Milestone 1 minimum for the 10-node validation gate. The full 30-40 method target is the Milestone 2 scope. Priority:

**Milestone 1 Critical Methods (15–20)**:
- `getNodeParameter`, `getInputData`, `getInputSourceData`
- `helpers.request`, `helpers.requestWithAuthentication`, `helpers.httpRequest`
- `getCredentials`, `getWorkflowStaticData`
- `getExecutionId`, `getNode`, `getMode`
- `helpers.returnJsonArray`, `helpers.constructExecutionMetaData`
- `continueOnFail`, `getTimezone`

**Milestone 2 Extended Methods (remaining ~20)**:
- Binary data: `helpers.prepareBinaryData`, `helpers.getBinaryDataBuffer`, `helpers.copyBinaryFile`
- Pagination: `helpers.requestWithAuthenticationPaginated`
- Complex: `helpers.requestOAuth1`, `helpers.requestOAuth2`, `helpers.httpRequestWithAuthentication`
- Workflow: `getWorkflow`, `getRestApiUrl`, `getInstanceBaseUrl`

### n8n Version Compatibility

- Target: n8n v1.x `IExecuteFunctions` interface
- Document breaking changes from n8n 0.x → 1.x in compatibility matrix
- Pin to specific n8n-workflow package version in devDependencies

### Credential Mapping

Each n8n credential type maps to a Sinter secret scope:
- `oAuth2Api` → Sinter OAuth2 credential provider
- `httpHeaderAuth` → Sinter header-based secret
- Custom credential types → Sinter generic secret store with key mapping

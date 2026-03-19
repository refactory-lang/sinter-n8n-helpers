<p align="center">
  <a href="https://github.com/refactory-lang"><img src="https://raw.githubusercontent.com/refactory-lang/.github/main/assets/refactory-logo.svg" alt="Refactory" width="300"></a>
</p>

# sinter-n8n-helpers

TypeScript shadow library for n8n's `IExecuteFunctions` helper methods. Maps ~30-40 n8n runtime methods to Sinter runtime equivalents, enabling n8n node translation.

**Added in Refactory Supplement v0.3.** This is a one-time investment that unlocks the entire n8n node translation pipeline (see Sinter Proposal Section 8.3).

## Purpose

n8n nodes access runtime helpers via `this.getNodeParameter()`, `this.helpers.request()`, etc. These methods have no Rust equivalent — they're n8n-specific. This library provides Sinter-native replacements that expose the same TypeScript API.

## Key Methods

| n8n Method | Sinter Equivalent | Category |
|-----------|-------------------|----------|
| `this.getNodeParameter(name, i)` | `ctx.getParameter(name)` | Parameter access |
| `this.helpers.request(options)` | `ctx.httpRequest(options)` | HTTP |
| `this.helpers.requestWithAuthentication(...)` | `ctx.authenticatedRequest(...)` | Auth HTTP |
| `this.getInputData()` | `ctx.inputItems()` | Data flow |
| `this.getWorkflowStaticData(type)` | `ctx.workflowState(type)` | State |

## Usage in Normalization

The n8n helpers library is consumed during the Normalize-Det stage (idiomatic TS → constrained TS). The normalizer rewrites n8n method calls to sinter-n8n-helpers equivalents before Tier 1 translation to Rust.

## License

Apache-2.0

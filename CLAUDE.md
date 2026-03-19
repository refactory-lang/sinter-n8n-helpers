<!-- codemod-skill-discovery:begin -->
## Codemod Skill Discovery
This section is managed by `codemod` CLI.

- Core skill: `.agents/skills/codemod/SKILL.md`
- Package skills: `.agents/skills/<package-skill>/SKILL.md`
- List installed Codemod skills: `npx codemod agent list --harness antigravity --format json`

<!-- codemod-skill-discovery:end -->

## Project: sinter-n8n-helpers

TypeScript shadow library for n8n's `IExecuteFunctions` helper methods. Part of the [refactory-lang](https://github.com/refactory-lang) organization. Maps ~30-40 n8n runtime methods to Sinter runtime equivalents, enabling n8n node translation to Rust. Added in Refactory Supplement v0.3.

### Architecture

- **Source** (`src/`): Shadow implementations mapping n8n helpers to Sinter equivalents
- **Tests** (`tests/`): Test suite
- **Specs** (`specs/`): Implementation specifications (see `001-n8n-helper-shadows/`)

The library is consumed during the **Normalize-Det** stage (idiomatic TS -> constrained TS). The normalizer rewrites n8n method calls to sinter-n8n-helpers equivalents before Stage 1 translation to Rust.

### Running

```bash
npm install
npm test
```

### Key Files

| File | Purpose |
|------|---------|
| `src/` | n8n helper shadow implementations |
| `tests/` | Test suite |
| `specs/001-n8n-helper-shadows/` | Implementation specification |

### Key Method Mappings

| n8n Method | Sinter Equivalent |
|-----------|-------------------|
| `this.getNodeParameter(name, i)` | `ctx.getParameter(name)` |
| `this.helpers.request(options)` | `ctx.httpRequest(options)` |
| `this.helpers.requestWithAuthentication(...)` | `ctx.authenticatedRequest(...)` |
| `this.getInputData()` | `ctx.inputItems()` |
| `this.getWorkflowStaticData(type)` | `ctx.workflowState(type)` |

# Sollumz MCP Server Product Plan

## 1) Purpose
Create a production-grade MCP server for the Sollumz repository that exposes safe, scriptable GTA V asset workflows (import/export/validation/benchmark) to MCP clients.

This plan is designed to:
- Reuse existing Sollumz code paths and Blender operators.
- Minimize risk by starting from deterministic, testable workflows.
- Introduce security and reliability controls early.
- Provide an incremental delivery path from alpha to stable release.

---

## 2) Product Goals

### Primary goals
1. Expose high-value Sollumz workflows as MCP tools.
2. Return machine-readable outputs suitable for automation.
3. Ensure deterministic behavior in CI/non-interactive environments.
4. Keep execution safe (filesystem boundaries, timeouts, controlled subprocesses).

### Non-goals (v1)
- Rich interactive modeling sessions.
- Full UI mirroring of Blender panels.
- Remote multi-tenant hosting as a first release requirement.

---

## 3) Users and Use Cases

### Users
- Technical artists automating repetitive import/export operations.
- Build/release engineers running asset checks in CI.
- Tooling developers integrating asset pipelines into agentic workflows.

### v1 use cases
- "Import this asset and return created objects."
- "Export the selected/identified asset with explicit settings."
- "Run import/export benchmark and return structured timing results."
- "Validate that this asset can round-trip without hard failures."

---

## 4) Scope (v1)

### Required MCP tools
1. `ping`
2. `version_info`
3. `list_supported_formats`
4. `run_import_export_benchmark`
5. `import_asset`
6. `export_asset`
7. `validate_asset_basic`

### Optional (if time permits)
- `convert_asset` (format-to-format pipeline wrapper)
- `batch_import_export`
- `inspect_asset_metadata`

---

## 5) Technical Strategy

### 5.1 Runtime model
- Use a Python MCP server process as the front-end.
- Execute Blender-dependent workflows in subprocesses.
- Prefer short-lived execution workers for deterministic state.

### 5.2 Code organization (proposed)

```text
mcp_server/
  __init__.py
  server.py                  # MCP entrypoint/tool registration
  schemas.py                 # request/response models
  errors.py                  # error classes + MCP error mapping
  policy.py                  # path allowlist / limits
  blender_runner.py          # subprocess execution helpers
  adapters/
    perf_ie_adapter.py       # wrapper for existing perf command
    import_export_adapter.py # wrappers for import/export operations
  tools/
    health.py
    formats.py
    benchmark.py
    import_asset.py
    export_asset.py
    validate_asset.py
tests_mcp/
  test_schemas.py
  test_policy.py
  test_tools_health.py
  test_tools_benchmark.py
  test_tools_import_export.py
```

### 5.3 Data contracts
- Use strict schema validation for every tool input/output.
- Standardize response envelope:
  - `ok: bool`
  - `data: object | null`
  - `error: { code, message, details } | null`
  - `artifacts: list[path]`

### 5.4 Security and policy
- Restrict filesystem operations to configured roots.
- Sanitize all client-provided paths.
- Enforce per-tool timeout and max batch size.
- Never allow arbitrary Python execution from client input.

---

## 6) Implementation Phases

## Phase 0 — Design and contract freeze (2 days)
**Deliverables**
- Tool list and JSON-like schemas.
- Error code taxonomy.
- Policy defaults (path roots, timeout values).

**Exit criteria**
- All v1 tool signatures approved.
- Risks documented with mitigations.

## Phase 1 — Server foundation (3 days)
**Deliverables**
- `mcp_server/server.py` skeleton with tool registry.
- Shared request parsing + response envelope.
- `ping` and `version_info` tools.

**Exit criteria**
- Local MCP client can call health tools successfully.

## Phase 2 — Existing CLI integration first (3 days)
**Deliverables**
- `run_import_export_benchmark` tool via adapter to current performance workflow.
- Parse and return structured benchmark output.

**Exit criteria**
- Tool output stable across repeated runs on test assets.

## Phase 3 — Import/export tools (5 days)
**Deliverables**
- `import_asset` and `export_asset` tools.
- Deterministic object selection/state handling.
- Artifact path reporting.

**Exit criteria**
- Import then export works for representative `.ydr` and `.yft` examples.

## Phase 4 — Validation and hardening (4 days)
**Deliverables**
- `validate_asset_basic` checks.
- Timeout/retry/error mapping.
- Structured logs and trace IDs.

**Exit criteria**
- Failure modes are readable and actionable.

## Phase 5 — Documentation and alpha release (2 days)
**Deliverables**
- User/operator docs.
- Examples of MCP client config and calls.
- Changelog + alpha release tag.

**Exit criteria**
- New user can run a full example without implicit tribal knowledge.

---

## 7) Milestones

### Milestone A (end of week 1)
- Foundation + benchmark tool complete.

### Milestone B (end of week 2)
- Import/export tools complete.

### Milestone C (end of week 3)
- Validation + hardening complete.

### Milestone D
- Alpha release + feedback loop.

---

## 8) Testing Strategy

### 8.1 Unit tests
- Schema validation edge cases.
- Error mapping and policy enforcement.
- Path normalization and rejection tests.

### 8.2 Integration tests
- End-to-end tool calls on known test assets.
- Success and failure path verification.
- Artifact generation checks.

### 8.3 CI gates
- Lint/format checks.
- Unit tests required.
- Integration tests on Blender-capable runner.

---

## 9) Observability

- Structured logging with request IDs.
- Log levels: INFO for lifecycle, WARN for recoverable issues, ERROR for failures.
- Optional timing metrics per tool (latency, success/failure count).

---

## 10) Risk Register

1. **Blender startup overhead**
   - Mitigation: benchmark cold/hot starts, optionally add pooled workers later.

2. **Operator context fragility**
   - Mitigation: reset scene state before each tool; explicit precondition checks.

3. **Asset variability and edge cases**
   - Mitigation: start with known fixture corpus, expand continuously.

4. **Path and environment differences across platforms**
   - Mitigation: normalize paths and enforce policy centrally.

5. **Contract drift as tools evolve**
   - Mitigation: schema versioning and compatibility notes.

---

## 11) Release Plan

### Alpha (v0.1)
- health + benchmark + import/export tools
- docs + examples

### Beta (v0.2)
- validation tool improvements
- stronger diagnostics + better error taxonomy

### Stable (v1.0)
- finalized schemas
- migration notes for any breaking changes

---

## 12) Definition of Done (v1)

- All required tools implemented and documented.
- Integration tests passing on target CI runner.
- Clear operator runbook for installation and troubleshooting.
- Security constraints and defaults enabled by default.
- Versioned API contracts published.

---

## 13) First 10 Engineering Tasks

1. Create `mcp_server/` package with `server.py` and `schemas.py`.
2. Add base response/error envelope utilities.
3. Add `ping` tool.
4. Add `version_info` tool.
5. Implement policy module with path allowlist and timeout config.
6. Build Blender subprocess runner helper.
7. Add benchmark adapter based on existing import/export perf flow.
8. Add `run_import_export_benchmark` tool + tests.
9. Add `import_asset` and `export_asset` tool stubs + schema tests.
10. Create initial user docs with sample requests/responses.

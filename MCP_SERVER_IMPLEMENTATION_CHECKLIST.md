# Sollumz MCP Server Implementation Checklist

Use this checklist to execute the product plan end-to-end.

## A. Planning and Design
- [ ] Confirm v1 tools and non-goals.
- [ ] Define request/response schemas for each tool.
- [ ] Define standard error codes and remediation guidance.
- [ ] Define policy defaults (path roots, timeout, max batch size).

## B. Repository Setup
- [ ] Create `mcp_server/` package structure.
- [ ] Add `tests_mcp/` directory and baseline test files.
- [ ] Add dev entrypoint command for local MCP execution.

## C. Core Server
- [ ] Implement tool registry and dispatch.
- [ ] Implement envelope helpers (`ok`, `data`, `error`, `artifacts`).
- [ ] Implement structured logging with request IDs.

## D. Security/Policy
- [ ] Implement path normalization and allowlist checks.
- [ ] Implement timeout limits per tool.
- [ ] Reject unknown/untrusted arguments.
- [ ] Add tests for policy violations.

## E. Tool Delivery (v1)
- [ ] `ping`
- [ ] `version_info`
- [ ] `list_supported_formats`
- [ ] `run_import_export_benchmark`
- [ ] `import_asset`
- [ ] `export_asset`
- [ ] `validate_asset_basic`

## F. Testing
- [ ] Unit tests for schema validation.
- [ ] Unit tests for error mapping.
- [ ] Integration tests for benchmark tool.
- [ ] Integration tests for import/export tools.
- [ ] Add regression tests for discovered edge cases.

## G. Documentation
- [ ] Add user docs for setup and usage.
- [ ] Add API/tool docs with sample payloads.
- [ ] Add troubleshooting guide (common failures).
- [ ] Add release notes for alpha.

## H. Release Readiness
- [ ] CI checks pass.
- [ ] Manual smoke run from clean environment.
- [ ] Version and changelog updated.
- [ ] Tag and publish alpha release.

## I. Post-Alpha Follow-up
- [ ] Collect feedback/issues.
- [ ] Prioritize fixes and ergonomic improvements.
- [ ] Reassess architecture for performance optimization.

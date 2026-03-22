# MCP Inspector Documentation

This directory contains comprehensive documentation for the MCP Inspector assessment tool.

---

## Quick Start

| Document                                          | Purpose                                               |
| ------------------------------------------------- | ----------------------------------------------------- |
| [CLI Assessment Guide](CLI_ASSESSMENT_GUIDE.md)   | Three CLI modes, configuration, CI/CD integration     |
| [Architecture & Value](ARCHITECTURE_AND_VALUE.md) | What inspector-assessment provides and why it matters |

---

## Programmatic API

Integration documentation for using the npm package programmatically.

| Document                                                    | Purpose                                          |
| ----------------------------------------------------------- | ------------------------------------------------ |
| [Public API Surface](PUBLIC_API.md)                         | Stable exports, entry points, versioning policy  |
| [Programmatic API Guide](PROGRAMMATIC_API_GUIDE.md)         | Step-by-step guide for AssessmentOrchestrator    |
| [API Reference](API_REFERENCE.md)                           | Complete API documentation and method signatures |
| [Integration Guide](INTEGRATION_GUIDE.md)                   | Practical patterns for multi-server, CI/CD, etc. |
| [Type Reference](TYPE_REFERENCE.md)                         | Complete TypeScript type reference               |
| [Assessment Types Import](ASSESSMENT_TYPES_IMPORT_GUIDE.md) | Modular structure, import patterns, tree-shaking |

---

## Logging & Diagnostics

| Document                                        | Purpose                                             |
| ----------------------------------------------- | --------------------------------------------------- |
| [Logging Guide](LOGGING_GUIDE.md)               | Structured logging configuration, CLI flags, levels |
| [CLI Assessment Guide](CLI_ASSESSMENT_GUIDE.md) | Logging section with output examples                |

---

## Core Assessment

### Assessment Modules

| Document                                                                  | Purpose                                                    |
| ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [Assessment Catalog](ASSESSMENT_CATALOG.md)                               | Complete assessment module reference and tier organization |
| [Assessment Module Developer Guide](ASSESSMENT_MODULE_DEVELOPER_GUIDE.md) | Creating and extending assessment modules                  |
| [Scoring Algorithm Guide](SCORING_ALGORITHM_GUIDE.md)                     | Module weights, thresholds, calculations                   |
| [Assessment Modules API](ASSESSMENT_MODULES_API.md)                       | Module API surface and interfaces                          |
| [Assessment Modules Integration](ASSESSMENT_MODULES_INTEGRATION.md)       | Integrating modules into the orchestrator                  |

### Error Handling

| Document                                                    | Purpose                                    |
| ----------------------------------------------------------- | ------------------------------------------ |
| [Error Handling Conventions](ERROR_HANDLING_CONVENTIONS.md) | Error handling patterns and MCP compliance |

### Test Data Generation

| Document                                            | Purpose                                       |
| --------------------------------------------------- | --------------------------------------------- |
| [Test Data Architecture](TEST_DATA_ARCHITECTURE.md) | Core architecture, field handlers, boundaries |
| [Test Data Scenarios](TEST_DATA_SCENARIOS.md)       | Scenario categories, tool-aware generation    |
| [Test Data Extension](TEST_DATA_EXTENSION.md)       | Adding handlers, debugging, integration       |

### Response Validation

| Document                                                          | Purpose                                      |
| ----------------------------------------------------------------- | -------------------------------------------- |
| [Response Validation Core](RESPONSE_VALIDATION_CORE.md)           | Validation logic, business error detection   |
| [Response Validation Extension](RESPONSE_VALIDATION_EXTENSION.md) | Adding rules, troubleshooting, API reference |

### Progressive Testing

| Document                                                        | Purpose                  |
| --------------------------------------------------------------- | ------------------------ |
| [Progressive Complexity Guide](PROGRESSIVE_COMPLEXITY_GUIDE.md) | 2-level testing approach |

### Tool Analysis (Issue #57)

| Document                                                        | Purpose                                               |
| --------------------------------------------------------------- | ----------------------------------------------------- |
| [Architecture Detection Guide](ARCHITECTURE_DETECTION_GUIDE.md) | Server infrastructure analysis (12 DBs, 3 transports) |
| [Behavior Inference Guide](BEHAVIOR_INFERENCE_GUIDE.md)         | Multi-signal tool behavior classification             |

### Performance Tuning (Issue #37)

| Document                                                | Purpose                                |
| ------------------------------------------------------- | -------------------------------------- |
| [Performance Tuning Guide](PERFORMANCE_TUNING_GUIDE.md) | 7 tunable parameters, presets, configs |
| [Example Configs](examples/)                            | Ready-to-use performance config files  |

### Testing

| Document                                                  | Purpose                                  |
| --------------------------------------------------------- | ---------------------------------------- |
| [Test Utilities Reference](TEST_UTILITIES_REFERENCE.md)   | Mock factory API for assessment testing  |
| [Test Organization Pattern](TEST_ORGANIZATION_PATTERN.md) | Split test file conventions and patterns |

### Lessons Learned

Documented patterns and practices from real-world development:

| Document                                                                                | Purpose                                       |
| --------------------------------------------------------------------------------------- | --------------------------------------------- |
| [Lessons Learned Index](lessons-learned/README.md)                                      | Navigation hub for all lessons learned        |
| [Type-Safe Testing Patterns](lessons-learned/type-safe-testing-patterns.md)             | Patterns from Issue #186 (189 any type fixes) |
| [Test Automator Implementation](lessons-learned/test-automator-implementation-guide.md) | Guide for automated test generation           |

---

## JSONL Events API

Real-time progress streaming for CLI/auditor integration.

| Document                                     | Purpose                                   |
| -------------------------------------------- | ----------------------------------------- |
| [Event Reference](JSONL_EVENTS_REFERENCE.md) | All 13 event types and schema definitions |
| [Algorithms](JSONL_EVENTS_ALGORITHMS.md)     | EventBatcher and AUP enrichment           |
| [Integration](JSONL_EVENTS_INTEGRATION.md)   | Lifecycle examples, checklist, testing    |

---

## Security Testing

| Document                                                  | Purpose                                         |
| --------------------------------------------------------- | ----------------------------------------------- |
| [Security Patterns Catalog](SECURITY_PATTERNS_CATALOG.md) | Comprehensive attack patterns and payloads      |
| [Testbed Setup Guide](TESTBED_SETUP_GUIDE.md)             | A/B validation with vulnerable-mcp/hardened-mcp |
| [Security Audits](security/)                              | Security audit reports                          |

---

## Specifications

| Document                                          | Purpose                            |
| ------------------------------------------------- | ---------------------------------- |
| [Manifest Requirements](MANIFEST_REQUIREMENTS.md) | manifest_version 0.3 specification |
| [MCP Spec Reference](mcp_spec_11-2025.md)         | MCP protocol revision 2025-11-25   |

---

## Maintenance & Operations

### Deprecation Documentation (v1.25.2+)

Start with [Deprecation Index](DEPRECATION_INDEX.md) for navigation, then choose:

| Document                                                            | Purpose                                        |
| ------------------------------------------------------------------- | ---------------------------------------------- |
| [Deprecation Index](DEPRECATION_INDEX.md)                           | Navigation hub for all deprecation docs        |
| [Deprecation Guide](DEPRECATION_GUIDE.md)                           | What's deprecated, why, and migration timeline |
| [Deprecation API Reference](DEPRECATION_API_REFERENCE.md)           | Technical reference, warning formats, testing  |
| [Deprecation Migration Examples](DEPRECATION_MIGRATION_EXAMPLES.md) | Copy-paste ready code examples                 |
| [Deprecation Removal Checklist](DEPRECATION_REMOVAL_CHECKLIST.md)   | Checklist for removing deprecated code         |

### General Maintenance

| Document                                            | Purpose                                |
| --------------------------------------------------- | -------------------------------------- |
| [Upstream Sync Workflow](UPSTREAM_SYNC_WORKFLOW.md) | Sync procedure with upstream inspector |

---

## Security Practices

### Docker Isolation

- **`build-client:docker`** (package.json): Uses `docker run --rm -v ... node:22-slim` to build the Vite client. Required because macOS sandbox kills unsigned native binaries (esbuild, rollup). The container is ephemeral (`--rm`), uses a volume mount for build artifacts, and runs no privileged operations.
- **`assess-dvmcp-all.sh`**: Runs the DVMCP (Damn Vulnerable MCP) testbed in a Docker container with port mapping. Docker provides network and filesystem isolation for the intentionally vulnerable server. The container is user-managed (`docker run -d`).

### Credential Handling

- **GitHub Actions** (`.github/actions/code-review/`): Uses `ANTHROPIC_API_KEY` and `GITHUB_TOKEN` via GitHub Actions secrets. These are injected as environment variables, never logged, and scoped to the workflow run.
- **CLI credential filtering** (`cli/src/cli.ts`): The CLI filters environment variables matching `_API_KEY`, `_SECRET`, `_TOKEN`, `_PASSWORD`, `_CREDENTIAL` suffixes before passing env to child processes. This prevents accidental credential leakage to MCP server subprocesses.

### External Code Execution

- Assessment tools connect to MCP servers but do not execute server code directly. Server processes are started externally (by the user or CI) before the inspector connects.
- `npm install` in scripts (`update-version.js`) runs against the local project only, not external code.

---

## Base Inspector Reference

Documentation for the underlying MCP Inspector that this assessment tool builds upon:

| Document                                            | Purpose                                           |
| --------------------------------------------------- | ------------------------------------------------- |
| [Base Inspector Guide](BASE_INSPECTOR_GUIDE.md)     | UI operation, Docker, auth, transports, config    |
| [Fork History](FORK_HISTORY.md)                     | Upstream relationship, sync status, what we added |
| [Upstream Sync Workflow](UPSTREAM_SYNC_WORKFLOW.md) | Sync procedure with upstream inspector            |

---

---

**Last Updated**: 2026-03-22

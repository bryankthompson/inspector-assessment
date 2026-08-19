# MCP Inspector Assessment (Experimental, Archived)

An experimental fork of [Anthropic's MCP Inspector](https://github.com/modelcontextprotocol/inspector) that explored automated stress-testing and security assessment of MCP servers.

## Status: Archived

This project is no longer actively developed. It served as an experimental exploration of whether automated injection testing could effectively validate MCP server security. The findings from this work informed subsequent approaches to MCP server review.

## What This Was

This project extended the MCP Inspector with an automated assessment framework that attempted to validate MCP servers by:

- Connecting to a running server and invoking each tool with injection payloads (command injection, SQL injection, path traversal, SSRF, code execution)
- Analyzing tool responses for evidence of actual code execution vs safe data reflection
- Scoring servers across functionality, security, error handling, documentation, and protocol compliance
- Providing a CLI for CI/CD integration alongside the web UI

The assessment engine included 18 modules, 31 attack patterns, temporal/rug-pull detection, cross-tool chain testing, and a weighted scoring rubric.

## What We Learned

**The core hypothesis was that automated injection testing could catch real MCP server vulnerabilities.** After extensive testing — including building a dedicated vulnerability testbed — we found that this approach has fundamental limitations:

1. **The real attack surface is the LLM, not the tool's input handler.** MCP exploits primarily happen through the model's interaction with tool descriptions and outputs — prompt injection via hidden instructions in descriptions, context poisoning through tool outputs, and social engineering through misleading metadata. Sending `whoami` to a tool doesn't test any of this.

2. **Injection testing only catches what it's designed for.** The testbed tools were purpose-built to be vulnerable to injection payloads (`eval()`, `subprocess.run(shell=True)`). Real-world MCP servers rarely have these obvious patterns, and the ones that do are easily caught by static code analysis without needing to invoke tools.

3. **AI-based review outperforms automated testing for this domain.** Having an LLM read tool descriptions, source code, and manifests — and judge them holistically — catches a broader class of issues (AUP violations, misleading descriptions, undisclosed telemetry, social engineering) that automated testing structurally cannot detect.

4. **Behavioral testing adds proof to findings already made by static analysis.** In testing against a 59-tool vulnerability testbed, every tool flagged by injection testing was also identifiable from source code patterns. The behavioral evidence was confirmatory, not additive.

These findings led to a shift toward AI-based review pipelines for MCP server validation, where the model evaluates servers the way it would actually interact with them.

## Repository Structure

This is a monorepo with three workspaces:

- `client/` — React frontend (Vite, TypeScript, Tailwind)
- `server/` — Express backend (TypeScript)
- `cli/` — Command-line assessment runner (TypeScript)

**Original Repository**: https://github.com/modelcontextprotocol/inspector

## Running Locally

If you want to explore the assessment framework:

```bash
git clone https://github.com/bryankthompson/inspector-assessment.git
cd inspector-assessment
npm install
npm run build
npm run dev          # Web UI at http://localhost:6274
```

CLI assessment:

```bash
npm run assess -- --server <name> --config <path-to-config.json>
```

## Acknowledgments

This project builds on the foundation provided by Anthropic's MCP Inspector team. The original inspector remains the recommended tool for MCP server debugging and development: https://github.com/modelcontextprotocol/inspector

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

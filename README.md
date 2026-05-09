# nuk

OWASP MCP Top 10 security scanner. Rust.

## What it does

Scans MCP server source code for security issues. 4-layer analysis: regex patterns → AST parsing (tree-sitter) → taint tracking → cross-file call graph.

Also does: remote endpoint probing, npm/GitHub discovery, runtime proxy with injection blocking.

## Install

Download from [Releases](https://github.com/nuk-scan/nuk/releases):

```
# Linux
curl -L https://github.com/nuk-scan/nuk/releases/latest/download/nuk-linux-x86_64 -o nuk
chmod +x nuk

# macOS (Apple Silicon)
curl -L https://github.com/nuk-scan/nuk/releases/latest/download/nuk-darwin-arm64 -o nuk
chmod +x nuk

# macOS (Intel)
curl -L https://github.com/nuk-scan/nuk/releases/latest/download/nuk-darwin-x86_64 -o nuk
chmod +x nuk
```

## Usage

```bash
# Scan by GitHub URL
nuk analyze --path https://github.com/user/mcp-server

# Scan by shorthand
nuk analyze --path user/mcp-server

# Scan local directory
nuk analyze --path ./my-mcp-server

# Remote endpoint scan
nuk scan --target https://mcp.example.com/sse

# Discover MCP servers on npm
nuk discover --source npm

# Runtime proxy
nuk proxy --listen 127.0.0.1:9100 --upstream https://mcp-server:8080

# Output formats
nuk analyze --path ./server --format json
nuk analyze --path ./server --format sarif
```

## Scan Report

We scanned 1,470 npm MCP packages and 37 GitHub MCP projects.

→ [Full report](https://nuk-scan.github.io/nuk/)

Summary:
- npm: 34% F-grade (506 out of 1,470)
- GitHub: 78% F-grade (29 out of 37)
- Total Critical findings: 11,252
- Dataflow paths detected: 5,583
- Cross-file attack chains: 3,443

Raw data: [`scan-results/verified-mcp-servers.csv`](scan-results/verified-mcp-servers.csv)

## Analysis Layers

| Layer | What | How |
|-------|------|-----|
| 1 | Pattern matching | YAML rules, OWASP MCP01–10, regex + FP filters |
| 2 | AST parsing | tree-sitter (TS, Python, Rust), confidence levels |
| 3 | Taint tracking | Multi-pass iterative, input → sink, sanitizer-aware |
| 4 | Cross-file | Import resolution, call graph BFS, 5-depth max |

## OWASP MCP Top 10

| ID | Threat |
|----|--------|
| MCP01 | Token & Credential Mismanagement |
| MCP02 | Excessive Privilege Grants |
| MCP03 | Tool Poisoning Attacks |
| MCP04 | Supply Chain Compromise |
| MCP05 | Command & Code Injection |
| MCP06 | Intent Mismatch & Prompt Injection |
| MCP07 | Authentication & Authorization Gaps |
| MCP08 | Audit & Telemetry Gaps |
| MCP09 | Shadow MCP Servers |
| MCP10 | Context Injection & Oversharing |

## Runtime Proxy

```bash
nuk proxy --listen 127.0.0.1:9100 --upstream https://mcp-server:8080
```

Sits between client and MCP server. Inspects every JSON-RPC message:
- Blocks shell injection, SQL injection, path traversal, SSRF
- Blocks prompt injection in tool arguments
- Detects sensitive data leakage in responses
- Rate limiting
- Tool description drift detection (rug-pull)
- Full audit log (JSONL)

## Custom Rules

Rules are YAML files in `rules/`. Add your own:

```yaml
id: CUSTOM01
name: My Custom Rule
severity_default: HIGH
description: Detects something specific
patterns:
  - regex: "dangerous_pattern"
    label: Found dangerous pattern
    severity: CRITICAL
remediation: Fix it by doing X
```

## License

Engine binary: proprietary.
OWASP rules (`rules/`): Apache 2.0.
Scan results data: CC BY 4.0.

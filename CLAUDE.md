# CLAUDE.md - AI Assistant Guide for Crush

## Project Overview

Crush is a terminal-based AI coding assistant from [Charm](https://charm.sh). It provides an interactive CLI for working with various LLMs (Anthropic, OpenAI, Google, etc.) with support for LSPs, MCPs, and session management.

**Repository**: `github.com/charmbracelet/crush`
**Version**: 0.16.1 (latest tag)
**License**: FSL-1.1-MIT

## Tech Stack

- **Language**: Go 1.25.0
- **CLI Framework**: Cobra
- **TUI Framework**: Bubble Tea v2 (beta), Lip Gloss v2 (beta)
- **Database**: SQLite (go-sqlite3)
- **LLM Providers**: Anthropic, OpenAI, Google (Gemini/Vertex AI), AWS Bedrock, Azure OpenAI, and more
- **Protocols**: MCP (Model Context Protocol), LSP (Language Server Protocol)
- **Build**: Task (Taskfile), GoReleaser
- **Linting**: golangci-lint with gofumpt

## Project Structure

```
crush/
├── main.go                 # Entry point
├── go.mod / go.sum         # Dependencies
├── Taskfile.yaml           # Task runner config
├── .golangci.yml           # Linter configuration
├── .goreleaser.yml         # Release configuration
├── schema.json             # JSON schema for config
├── sqlc.yaml               # SQL code generation config
├── internal/               # Private packages
│   ├── agent/              # LLM agent logic & tools
│   │   ├── tools/          # Built-in tools (edit, view, grep, etc.)
│   │   ├── prompt/         # System prompts
│   │   └── templates/      # Prompt templates
│   ├── app/                # Application logic
│   ├── cmd/                # CLI commands
│   ├── config/             # Configuration loading/parsing
│   ├── db/                 # SQLite database & migrations
│   ├── event/              # Metrics/analytics events
│   ├── format/             # Output formatting
│   ├── fsext/              # Filesystem utilities
│   ├── history/            # Command history
│   ├── home/               # Home directory utilities
│   ├── log/                # Logging
│   ├── lsp/                # LSP client
│   ├── message/            # Message types
│   ├── permission/         # Tool permission system
│   ├── pubsub/             # Event pub/sub
│   ├── session/            # Session management
│   ├── shell/              # Shell command execution
│   ├── term/               # Terminal utilities
│   ├── tui/                # Terminal UI components
│   │   ├── components/     # UI components
│   │   ├── exp/            # Experimental components
│   │   └── page/           # TUI pages
│   └── version/            # Version info
├── scripts/                # Utility scripts
└── .github/                # GitHub Actions workflows
```

## Key Files

| File | Purpose |
|------|---------|
| `main.go` | Application entry point |
| `internal/cmd/root.go` | Root CLI command |
| `internal/cmd/run.go` | Main run command |
| `internal/agent/agent.go` | LLM agent core (884 lines) |
| `internal/agent/coordinator.go` | Agent coordination (758 lines) |
| `internal/config/config.go` | Configuration types |
| `internal/config/load.go` | Config loading logic |
| `internal/tui/tui.go` | Main TUI logic (691 lines) |
| `internal/app/app.go` | Application state |
| `internal/shell/shell.go` | Shell execution |

## Development Setup

### Prerequisites
- Go 1.25+ (uses experimental greenteagc)
- Task (https://taskfile.dev)
- golangci-lint

### Build & Run

```bash
# Build
task build

# Run directly
task run

# Run with arguments
task run -- --debug

# Install to GOPATH/bin
task install

# Run with profiling
task dev
```

### Environment Variables

```bash
# LLM Provider Keys
ANTHROPIC_API_KEY
OPENAI_API_KEY
OPENROUTER_API_KEY
GEMINI_API_KEY
GROQ_API_KEY

# AWS Bedrock
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION

# Azure OpenAI
AZURE_OPENAI_API_ENDPOINT
AZURE_OPENAI_API_KEY

# Vertex AI
VERTEXAI_PROJECT
VERTEXAI_LOCATION

# Development
CRUSH_PROFILE=true      # Enable pprof at localhost:6060
CRUSH_DISABLE_METRICS=1 # Disable analytics

# Build
CGO_ENABLED=0
GOEXPERIMENT=greenteagc
```

## Common Commands

```bash
# Build & Run
task build              # Build binary
task run                # Run with go run
task install            # Install to GOPATH/bin
task dev                # Run with profiling

# Testing
task test               # Run all tests
task test -- -v         # Verbose tests
task test -- -run=Name  # Run specific test

# Code Quality
task lint               # Run linters
task lint:fix           # Run linters with auto-fix
task fmt                # Format with gofumpt

# Profiling
task profile:cpu        # 10s CPU profile
task profile:heap       # Heap profile
task profile:allocs     # Allocations profile

# Release
task schema             # Generate JSON schema
task release            # Create and push version tag
```

## Testing

- **Framework**: Go standard testing
- **Pattern**: `*_test.go` files alongside source

### Running Tests

```bash
# All tests
task test

# Specific package
go test ./internal/shell/...

# With race detection
go test -race ./...

# With coverage
go test -cover ./...
```

### Test Files
Tests are colocated with source files. Key test files:
- `internal/config/load_test.go` (1249 lines)
- `internal/agent/agent_test.go`
- `internal/shell/shell_test.go`
- `internal/lsp/client_test.go`

## Code Conventions

### Package Layout
- All application code in `internal/` (private packages)
- One responsibility per package
- Tests colocated with source

### Formatting & Linting
- **Formatter**: gofumpt (stricter than gofmt)
- **Imports**: goimports
- **Linter**: golangci-lint v2

### Disabled Linters
The following linters are disabled in `.golangci.yml`:
- `errcheck` - unchecked errors
- `ineffassign` - ineffectual assignments
- `unused` - unused code

### Code Style
- Use `slog` for structured logging
- Context-aware functions where appropriate
- Bubble Tea patterns for TUI components

### Configuration Files
- JSON format with `$schema` support
- Location priority: `.crush.json` > `crush.json` > `~/.config/crush/crush.json`

## When Making Changes

1. **Follow existing patterns** - Check similar code in the same package
2. **Add tests** - Create `*_test.go` files for new functionality
3. **Run linters** - `task lint` before committing
4. **Format code** - `task fmt` to run gofumpt
5. **Check builds** - `task build` to verify compilation
6. **Update schema** - Run `task schema` if config types change

### Adding a New Tool
1. Create file in `internal/agent/tools/`
2. Implement the tool interface
3. Register in tool initialization
4. Add tests

### Adding a New Provider
1. Update `internal/config/config.go` with provider types
2. Add provider logic in appropriate package
3. Update Catwalk (external repo) for default models

## Issues & Recommendations

### Critical Issues

1. **Future Go Version (go 1.25.0)**
   - `go.mod` specifies Go 1.25.0 which is unreleased/experimental
   - May cause build issues for contributors
   - Recommendation: Document Go version requirements clearly

2. **Disabled Security Linters**
   - `gosec` is commented out in `.golangci.yml`
   - Command injection risks exist (shell execution in `internal/shell/`)
   - Recommendation: Enable `gosec` and audit shell execution paths

3. **Disabled Error Checking**
   - `errcheck`, `ineffassign`, `unused` disabled
   - Could hide bugs and resource leaks
   - Recommendation: Enable these linters and fix violations

### Medium Priority

4. **Unfinished Work**
   - 30+ TODO/FIXME/HACK comments across 15 files
   - Key areas: `internal/agent/`, `internal/tui/`, `internal/config/`
   - Recommendation: Triage and address or create tracking issues

5. **Large Files**
   - `internal/config/load_test.go`: 1249 lines
   - `internal/agent/agent.go`: 884 lines
   - Recommendation: Consider splitting large files into smaller units

6. **Missing Documentation**
   - No CONTRIBUTING.md with contribution guidelines
   - No architecture documentation
   - Recommendation: Add contribution guide and architecture docs

### Low Priority

7. **Beta Dependencies**
   - Using Bubble Tea v2 and Lip Gloss v2 betas
   - API may change before stable release
   - Note: This is intentional for Charm's ecosystem

8. **Test Coverage**
   - No coverage reporting in CI
   - Recommendation: Add coverage reporting to build workflow

9. **Hardcoded Values**
   - Some configuration values hardcoded in code
   - Recommendation: Move to configuration or constants

### Security Notes

- Tool execution requires user permission by default
- `--yolo` flag bypasses all permission checks (dangerous)
- Shell commands executed via `mvdan.cc/sh` interpreter
- API keys stored in environment variables or config files

## External Resources

- **Documentation**: README.md (comprehensive)
- **Model Database**: [Catwalk](https://github.com/charmbracelet/catwalk)
- **Charm Organization**: [charm.sh](https://charm.sh)
- **Discord**: [charm.land/discord](https://charm.land/discord)

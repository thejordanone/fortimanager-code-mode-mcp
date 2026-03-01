# Contributing to FortiManager Code Mode MCP Server

Thank you for your interest in contributing! This guide will help you get started.

## Development Setup

### Prerequisites

- **Node.js** 20+ (LTS recommended)
- **npm** 9+
- **Git**

### Getting Started

```bash
# Clone the repository
git clone https://github.com/jmpijll/fortimanager-code-mode-mcp.git
cd fortimanager-code-mode-mcp

# Install dependencies
npm install

# Verify the build
npm run build
npm run lint
npm run typecheck
npm test  # 66 unit tests across 5 suites
```

### MCP Client Setup (VS Code Copilot)

To test the MCP server with VS Code Copilot:

```bash
# Build the project (compiles TypeScript + copies spec files)
npm run build

# Copy and configure MCP settings
cp .vscode/mcp.json.example .vscode/mcp.json
# Edit .vscode/mcp.json with your FortiManager details
```

Open Agent mode in VS Code Copilot Chat — the `search` and `execute` tools will appear automatically.

### Running Locally

```bash
# Create your environment config
cp .env.example .env
# Edit .env with your FortiManager details

# Development mode (hot reload)
npm run dev

# Or build and start
npm run build
npm start
```

## Code Standards

### TypeScript

- **Strict mode** is enabled — no `any`, use `unknown` and type guards
- **ESM modules** — all imports use `.js` extensions (TypeScript convention for ESM)
- **Explicit return types** on exported functions
- **Zod** for runtime validation of external inputs

### Naming Conventions

| Entity              | Convention             | Example                    |
| ------------------- | ---------------------- | -------------------------- |
| Files               | `kebab-case.ts`        | `fmg-client.ts`            |
| Classes             | `PascalCase`           | `FmgClient`                |
| Functions/variables | `camelCase`            | `rawRequest`               |
| Constants           | `SCREAMING_SNAKE_CASE` | `DEFAULT_EXECUTOR_OPTIONS` |
| Types/interfaces    | `PascalCase`           | `ExecuteResult`            |

### Error Handling

- Use custom error classes extending `Error`
- Always include context (URL, method, status code) in error messages
- Never swallow errors silently

### Formatting

This project uses **Prettier** for formatting and **ESLint** for linting:

```bash
# Format all files
npm run format

# Check formatting (CI)
npm run format:check

# Lint
npm run lint

# Lint with auto-fix
npm run lint:fix
```

## Testing

We use **Vitest** for testing. Tests live alongside source in `src/__tests__/`.

```bash
# Run all tests
npm test

# Watch mode
npm run test:watch

# With coverage
npm run test:coverage
```

### Writing Tests

- Co-locate tests in `src/__tests__/` with descriptive names (e.g., `client.test.ts`)
- Use fixtures from `src/__tests__/fixtures/` for sample data
- Mock external HTTP calls — never hit real FortiManager in unit tests
- Test files should import from the source using relative paths with `.js` extensions

### Integration Tests

The integration test suite (`scripts/live-test.ts`) validates both tools against a live FortiManager:

```bash
# Run integration tests (requires .env with FMG credentials)
npx tsx scripts/live-test.ts

# Include stress tests (200 sequential searches, 50 executes, heap check)
npx tsx scripts/live-test.ts --stress
```

### API Coverage Report

The spec coverage script (`scripts/spec-coverage.ts`) provides offline analysis and live URL validation:

```bash
# Offline spec analysis + cross-version comparison
npx tsx scripts/spec-coverage.ts

# Live URL validation against FortiManager (stratified sample)
npx tsx scripts/spec-coverage.ts --validate --sample 100
```

## Git Workflow

### Branch Naming

- `feat/<name>` — New features
- `fix/<name>` — Bug fixes
- `chore/<name>` — Maintenance (deps, CI, docs)

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>
```

**Types**: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `build`

**Scopes**: `client`, `spec`, `executor`, `server`, `docker`, `ci`, `docs`

**Examples**:

```
feat(client): add JSON-RPC request multiplexing
fix(executor): increase QuickJS memory limit to 32MB
chore(deps): update @modelcontextprotocol/sdk to 1.2.0
docs: add architecture diagram to README
```

### Pull Request Process

1. **Create a feature branch** from `main`
2. **Make changes** with conventional commits
3. **Ensure CI passes** — lint, typecheck, tests, build
4. **Push and open a PR** against `main`
5. **Describe your changes** using the PR template
6. **Squash merge** once approved

## Regenerating API Specs

If you update the HTML documentation files in `docs/api-reference/`, regenerate the JSON specs:

```bash
npm run generate:spec
```

This parses the FortiManager HTML API reference docs and produces:

- `src/spec/fmg-api-spec-7.4.json` (from 7.4.9 docs)
- `src/spec/fmg-api-spec-7.6.json` (from 7.6.5 docs)

> **Note**: Generated spec files are large (98–126 MB) and tracked via Git LFS. They are built offline and shipped with the Docker image. The `npm run build` command automatically copies them to `dist/spec/`.

## Reporting Bugs

- Open a [GitHub Issue](https://github.com/jmpijll/fortimanager-code-mode-mcp/issues) with the `bug` label
- Include: reproduction steps, expected vs actual behavior, FortiManager version, relevant logs
- Reference the code area in the title: `[client] Connection timeout on large responses`

## Security Vulnerabilities

Please report security issues privately. See [SECURITY.md](SECURITY.md) for details.

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).

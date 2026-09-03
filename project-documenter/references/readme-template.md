# README template

Use this as a guide for section structure, not as literal boilerplate — fill every section from what Phase 1 actually found, and drop sections that don't apply to this project rather than leaving them as stubs.

```markdown
# <Project Name>

<One or two sentences: what this project is and what problem it solves. Pull this from an existing description if there is one (package.json "description", pyproject.toml, an about section) — otherwise infer it from the entry point and core modules, and say so if it's inferred rather than stated.>

## Features

<Only include if there's a clear, non-generic list to give — 3-6 bullets on what the project actually does, not generic praise.>

## Prerequisites

<Everything needed before install: language/runtime version, package manager, required external services (database, cache, etc.), any accounts/API keys needed. Be specific about versions where the project specifies them.>

- Node.js >= 18 (see `engines` in package.json)
- PostgreSQL 14+
- ...

## Installation

<Exact, copy-pasteable commands, in order, verified against the manifest/lockfile/CI config — not generic "npm install" boilerplate unless that really is all it takes.>

```bash
git clone <repo-url>
cd <project-dir>
npm install
```

## Configuration

<Only if the project reads env vars or config files. List each required/optional variable, what it's for, and where to set it (`.env`, a config file, etc.). If a `.env.example` exists, mention it and keep this section in sync with it.>

| Variable | Required | Purpose |
|---|---|---|
| `DATABASE_URL` | Yes | Postgres connection string |
| `PORT` | No (default 3000) | HTTP server port |

## Running the project

<The actual commands to run it locally, and — if relevant — how to run it in production/Docker. Distinguish dev mode (hot reload, etc.) from a production start command if both exist.>

```bash
npm run dev      # local development, hot reload
npm run build && npm start   # production
```

## Testing

<Only if there's a real test setup. The actual command(s), and briefly what's covered (unit/integration/e2e) if that's discoverable.>

```bash
npm test
```

## Project structure

<A short top-level directory map with a one-line purpose for each — just enough to orient someone, with a pointer to docs/ARCHITECTURE.md for the full picture. Don't duplicate the whole architecture doc here.>

```
src/
  routes/       HTTP route definitions
  services/     business logic
  models/       database models
docs/           architecture and file-by-file reference — see docs/ARCHITECTURE.md
```

## Further documentation

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for how the project is structured internally and what each module does.

## License

<Only if a LICENSE file or license field exists — name it and link to the file. Don't invent a license.>
```

## Judgment calls

- If the project is a library/package meant to be imported rather than run, replace "Running the project" with a "Usage" section showing how to import and use it (a short real code example beats a description).
- If there's a CLI, show `--help` output or the actual flag list rather than prose.
- If install genuinely is just one command, don't pad it into several sections — a short README for a genuinely small project is correct, not incomplete.
- Never fabricate a license, a badge, a CI status, or a contributing-guide reference that doesn't exist in the repo.

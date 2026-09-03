# Analysis checklist

Goal: come out of this phase knowing the project well enough that you could onboard someone by talking, not guessing. Everything below should come from files you actually opened, commands you actually found, not from "projects like this usually...".

## 1. Detect the stack(s)

Look for manifest/lockfiles at the root and in any obvious sub-project directories. Don't infer the stack from file extensions alone — a `.py` file in a repo doesn't tell you if it's Django, Flask, a script, or a data pipeline.

| Signal file | Ecosystem | What to read from it |
|---|---|---|
| `package.json` | Node.js/JS/TS | `scripts` (real commands), `dependencies`/`devDependencies` (frameworks), `engines` (Node version) |
| `pyproject.toml`, `requirements.txt`, `Pipfile`, `setup.py` | Python | dependencies, Python version pin, entry points/`console_scripts` |
| `pom.xml`, `build.gradle`/`build.gradle.kts` | Java/Kotlin | dependencies, plugins, build/run tasks |
| `go.mod` | Go | module path, Go version, key deps |
| `Cargo.toml` | Rust | `[dependencies]`, `[[bin]]`/`[lib]` targets |
| `*.csproj`, `*.sln` | .NET | target framework, package refs |
| `Gemfile` | Ruby | gems, Ruby version |
| `composer.json` | PHP | dependencies, scripts |
| `Dockerfile`, `docker-compose.yml` | any | base image (tells you runtime/version), exposed ports, required services, env vars |
| CI config (`.github/workflows/*.yml`, `.gitlab-ci.yml`, etc.) | any | the *actual* install/build/test/lint commands the maintainers trust enough to run in CI — often the most reliable source of truth |
| `Makefile`, `Taskfile.yml`, `justfile` | any | task shortcuts that are usually the "real" commands developers use day to day |

A project can span more than one of these (e.g. a Python backend + a Node frontend, or a Go service with a Dockerfile). Document each stack you find.

## 2. Prerequisites

From what you found above, extract:
- Runtime/language version(s) required (exact pin if specified, otherwise minimum)
- Package manager (npm/pnpm/yarn — check for `package-lock.json` vs `pnpm-lock.yaml` vs `yarn.lock` to know which one; pip/poetry/uv; maven/gradle; etc.)
- External services the project needs to run (database, cache, message broker, cloud SDKs requiring credentials) — look in `docker-compose.yml`, config files, and code that opens connections (search for connection strings, `os.environ`/`process.env` reads, etc.)
- Required environment variables — grep for `process.env.`, `os.environ`, `os.getenv`, `System.getenv`, `ENV[`, config-loading libraries (dotenv, etc.), and check for a `.env.example`/`.env.sample` file

## 3. Real commands, not boilerplate

Pull install/build/run/test/lint commands from the actual files, in this priority order (higher = more trustworthy, since it's what the maintainers run):
1. CI workflow files
2. Makefile/Taskfile/justfile
3. `scripts` block in the manifest (`package.json` scripts, `pyproject.toml` tool config, etc.)
4. README if one already exists (verify it still matches the code before trusting it)

If a command can't be verified from any file (e.g. no scripts defined), say what the conventional command would be but don't present it as confirmed.

## 4. Entry points and control flow

Find where execution actually starts:
- Node: `main`/`bin` fields in `package.json`, or the file the primary `start` script points to
- Python: `if __name__ == "__main__"`, `console_scripts` entry points, a `manage.py`/`app.py`/`main.py`
- Java/Kotlin: the class with `public static void main`, or the Spring Boot `@SpringBootApplication` class
- Go: `func main()` in `package main`
- Rust: `src/main.rs` or `[[bin]]` targets
- Web frameworks: also note the routing entry (where URL routes are registered)

Trace a step or two from the entry point to understand the overall flow (e.g. "server starts → loads config → registers routes → routes dispatch to controllers → controllers call services → services hit the DB layer").

## 5. Directory structure and roles

`Glob` the top 1-2 levels of the repo and figure out what each top-level directory is for by sampling its contents — don't just guess from the name. A `services/` directory could mean microservices or a service-layer pattern; check which.

Note directories that are generated/vendored and should be excluded from documentation: `node_modules`, `.venv`/`venv`, `vendor`, `dist`/`build`/`out`, `.git`, `target` (Java/Rust), `bin`/`obj` (.NET), coverage reports, `__pycache__`.

## 6. File-by-file pass

For each significant file/module (see SKILL.md's definition of "significant"), note in a sentence or two:
- What it's responsible for
- What it depends on / what depends on it, if that relationship is important to understanding the architecture

Batch this with Grep/Glob to find files by pattern (routes, models, controllers, components) rather than opening every file one at a time when the codebase is large — read representative files in each category plus anything that looks like a hub (a router, a central config, a base class many things extend).

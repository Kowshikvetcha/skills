---
name: project-documenter
description: Generates or refreshes a project's README.md and a docs/ folder that explains overall architecture and the purpose of each significant file/module. Works across any language or stack (Node, Python, Java, Go, Rust, .NET, Ruby, PHP, etc.) by inspecting the actual repo rather than assuming a framework. Use this whenever the user asks to "document this project", "write a README", "explain the architecture", "add documentation", "create a docs folder", or similar — and also proactively suggest it (don't just wait to be asked) whenever you notice a repo has no README, a README that's clearly stale or a stub, or no architecture/docs folder while you're doing other work in it. Also trigger for "onboard a new developer to this codebase" or "explain how this project is structured" requests.
tools: Read, Glob, Grep, Bash, Edit, Write, AskUserQuestion
---

# Project Documenter

Produce two things for the project in the current working directory:

1. A comprehensive **README.md** at the project root — what the project is, what you need before you touch it, how to install and run it, and how to configure it.
2. A **docs/** folder that explains the architecture: how the pieces fit together, and what each significant file or module is for.

The hard part isn't the writing — it's the analysis. A README or architecture doc is only useful if it reflects what the code *actually* does, not what a typical project like this usually does. Never generate content from assumptions about the stack; read the real files first.

## Workflow

### Phase 1 — Analyze the project

Before writing anything, build an accurate picture of the project. See [references/analysis-checklist.md](references/analysis-checklist.md) for a language/ecosystem-agnostic checklist (how to detect the stack, find entry points, extract real install/run/test commands, and map the directory structure). Work through it directly with Glob/Grep/Read/Bash — don't skip to writing docs from a guess.

Key things you must come out of this phase with:
- The tech stack(s) actually in use (read manifest/lockfiles — `package.json`, `requirements.txt`/`pyproject.toml`, `pom.xml`/`build.gradle`, `go.mod`, `Cargo.toml`, `*.csproj`, `Gemfile`, `composer.json`, etc. — don't guess from file extensions alone)
- Real prerequisites and versions (runtime version pins, required services like a database or message queue, env vars the code reads)
- The actual install/build/run/test commands (pulled from scripts in the manifest, a Makefile, CI config, or Docker files — not generic boilerplate)
- Entry point(s) and how control flows from them
- The directory structure and what role each top-level directory plays
- For every significant file (not every file — see below), what it's responsible for

**What counts as "significant"**: files that define behavior or structure — modules, components, services, routes/controllers, models, config loaders, core scripts. Skip generated output, lockfiles, vendored/third-party code, build artifacts, and boilerplate config unless it's genuinely load-bearing for understanding the project. For a large repo, document at the directory/module level with representative key files called out, not literally every file — the goal is a map someone can navigate by, not an exhaustive inventory that goes stale the day someone adds a file.

If anything about purpose or intent is genuinely ambiguous from the code alone (e.g. why a particular service exists, a business rule that isn't self-evident), it's fine to ask the user rather than invent an explanation.

### Phase 2 — README.md

Read [references/readme-template.md](references/readme-template.md) for the section structure and what belongs in each part.

**If no README.md exists**, write one directly using the template as a guide, adapted to what Phase 1 actually found — omit sections that don't apply (e.g. no "Configuration" section if the project has no env vars or config files) rather than leaving placeholder text.

**If a README.md already exists**, do not overwrite it silently. Read it, compare it against what you learned in Phase 1, and:
1. Summarize for the user what's outdated, missing, or still accurate (e.g. "install steps are current; the run command changed from `npm start` to `npm run dev`; there's no mention of the `REDIS_URL` env var the code now requires").
2. Show the specific additions/changes you'd make (as a diff or clearly-labeled blocks).
3. Ask the user to confirm before applying anything.
4. Preserve sections that are clearly hand-written and still accurate (custom notes, team-specific context, badges, license text) — you're refreshing the factual/setup content, not rewriting their voice.

### Phase 3 — docs/ folder (architecture + file reference)

Read [references/architecture-template.md](references/architecture-template.md) for the structure to follow.

Decide the shape based on project size/complexity from Phase 1 — don't ask the user, just pick sensibly and say what you picked:
- **Small-to-medium projects** (roughly: a handful of top-level modules, code you could describe in one sitting): a single `docs/ARCHITECTURE.md` covering the high-level architecture plus a file-by-file / module-by-module reference table.
- **Large or multi-module projects** (multiple independently-deployable services, a monorepo with several packages, or simply too much ground for one file to stay readable): `docs/ARCHITECTURE.md` for the high-level system view (how the pieces talk to each other, key data/control flow), plus one file per major module or service under `docs/modules/<name>.md` for the detailed file-by-file breakdown of that area. Link from the top-level doc to each module doc.

Either way, the architecture doc should cover:
- A short description of the overall design/pattern (e.g. layered, MVC, microservices, monorepo with shared packages) — describe what's actually there, don't force it into a pattern name that doesn't fit
- How a request/task actually flows through the system, in enough detail that someone could trace it in the real code
- A directory/file reference: path → one or two sentences on its purpose and how it fits into the architecture. A table works well for this.
- Anything a new contributor would otherwise have to discover the hard way (non-obvious coupling, why something lives where it does, a directory that looks like dead code but isn't)

If `docs/` (or the specific files you're about to write) already exists with content, apply the same don't-clobber-silently approach as the README: summarize what's stale vs. current, show the proposed changes, and confirm before writing.

### Phase 4 — Wrap up

Tell the user what you wrote/updated and where, and flag anything you skipped because it needs their judgment (ambiguous purpose, a section you decided to leave out, a README/docs section you left alone because it's hand-maintained).

## Notes

- This is a documentation task, not a refactor — don't "fix" code you notice while reading it; note it to the user if it's worth mentioning, but stay in scope.
- If the project has sub-projects with their own manifests (a monorepo), treat each as part of Phase 1's stack detection rather than picking just one.
- Keep the tone of generated docs plain and factual. Avoid marketing language ("blazing fast", "seamless") unless it's quoting something the project itself claims.

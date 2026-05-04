# AGENTS

## Environment

- Python: .venv/bin/python (uv, CPython 3.13.3)
- GitHub auth for git/API is available via env vars: `GITHUB_USER`, `GITHUB_TOKEN` (PAT). Do not hardcode or commit tokens.
- For authenticated git over HTTPS in automation, use: `https://x-access-token:${GITHUB_TOKEN}@github.com/<owner>/<repo>.git`

## Deployment Split

- `ssh codex-usage` is the live host for both the stable and dev `codex-lb` runtimes.
- Production lives at `/opt/codex-lb` and runs the `codex-lb` container from `ghcr.io/soju06/codex-lb:latest`.
- Production follows the official upstream repo. Do not edit, rebuild, restart, or otherwise mutate production when working on branch changes from this checkout.
- Development lives at `/opt/codex-lb-dev` and uses the separate `codex-lb-dev` container plus the source checkout at `/opt/codex-lb-dev/source/codex-lb`.
- Standard dev deploy sequence from this workstation:
  - `git push origin <branch>`
  - `ssh codex-usage 'python3 /opt/codex-lb-dev/bin/manage.py rebuild --branch <branch>'`
  - `ssh codex-usage 'docker compose --project-directory /opt/codex-lb --file /opt/codex-lb/docker-compose.yml --profile dev ps'`
- If `manage.py rebuild` fails with a `codex-lb-dev` container-name conflict, clean up only the dev container and retry:
  - `ssh codex-usage 'docker rm -f codex-lb-dev && docker compose --project-directory /opt/codex-lb --file /opt/codex-lb/docker-compose.yml --profile dev up -d codex-lb-dev'`
- Verification commands:
  - `ssh codex-usage 'python3 /opt/codex-lb-dev/bin/manage.py status'`
  - `ssh codex-usage 'docker compose --project-directory /opt/codex-lb --file /opt/codex-lb/docker-compose.yml --profile dev ps'`
- Dev admin URL: `https://codex-lb-dev-admin.nosslin.dk/`
- Prod admin URL: `https://codex-lb-admin.nosslin.dk/`

## Code Conventions

The `/project-conventions` skill is auto-activated on code edits (PreToolUse guard).

| Convention | Location | When |
|-----------|----------|------|
| Code Conventions (Full) | `/project-conventions` skill | On code edit (auto-enforced) |
| Git Workflow | `.agents/conventions/git-workflow.md` | Commit / PR |

## Workflow (OpenSpec-first)

This repo uses **OpenSpec as the primary workflow and SSOT** for change-driven development.

### How to work (default)

1) Find the relevant spec(s) in `openspec/specs/**` and treat them as source-of-truth.
2) If the work changes behavior, requirements, contracts, or schema: create an OpenSpec change in `openspec/changes/**` first (proposal -> tasks).
3) Implement the tasks; keep code + specs in sync (update `spec.md` as needed).
4) Validate specs locally: `openspec validate --specs`
5) When done: verify + archive the change (do not archive unverified changes).

### Source of Truth

- **Specs/Design/Tasks (SSOT)**: `openspec/`
  - Active changes: `openspec/changes/<change>/`
  - Main specs: `openspec/specs/<capability>/spec.md`
  - Archived changes: `openspec/changes/archive/YYYY-MM-DD-<change>/`

## Documentation & Release Notes

- **Do not add/update feature or behavior documentation under `docs/`**. Use OpenSpec context docs under `openspec/specs/<capability>/context.md` (or change-level context under `openspec/changes/<change>/context.md`) as the SSOT.
- **Do not edit `CHANGELOG.md` directly.** Leave changelog updates to the release process; record change notes in OpenSpec artifacts instead.

### Documentation Model (Spec + Context)

- `spec.md` is the **normative SSOT** and should contain only testable requirements.
- Use `openspec/specs/<capability>/context.md` for **free-form context** (purpose, rationale, examples, ops notes).
- If context grows, split into `overview.md`, `rationale.md`, `examples.md`, or `ops.md` within the same capability folder.
- Change-level notes live in `openspec/changes/<change>/context.md` or `notes.md`, then **sync stable context** back into the main context docs.

Prompting cue (use when writing docs):
"Keep `spec.md` strictly for requirements. Add/update `context.md` with purpose, decisions, constraints, failure modes, and at least one concrete example."

### Commands (recommended)

- Start a change: `/opsx:new <kebab-case>`
- Create artifacts (step): `/opsx:continue <change>`
- Create artifacts (fast): `/opsx:ff <change>`
- Implement tasks: `/opsx:apply <change>`
- Verify before archive: `/opsx:verify <change>`
- Sync delta specs → main specs: `/opsx:sync <change>`
- Archive: `/opsx:archive <change>`

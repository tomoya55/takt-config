# takt-config

Shared [takt](https://github.com/nrslib/takt) workflow presets and persona configurations for AI agent coordination.

## Structure

```
takt-config/
├── config.yaml               # Default provider/model and persona_providers
├── facets/
│   ├── personas/
│   │   ├── planner.md         # Senior architect — plans phased milestones
│   │   ├── coder.md           # Full-stack developer — implements features
│   │   └── reviewer.md        # Pragmatic reviewer — blocks on real issues only
│   └── policies/
│       └── delivery-standards.md  # CI/CD first, PR sizing, merge-readiness
└── workflows/
    └── phased-delivery.yaml   # Plan → implement → review loop
```

## Persona → Provider Mapping

Configured in `~/.takt/config.yaml` under `persona_providers`:

| Persona | Provider | Model | Role |
|---------|----------|-------|------|
| planner | Claude Code | opus | Planning and milestone definition |
| coder | Cursor | composer-2-fast | Implementation and fixes |
| reviewer | Codex | codex | Code review (skips trivial findings) |

Workflows only reference personas by name — provider/model resolution is handled by `persona_providers`.

## Workflow Presets

### `phased-delivery`

Plan → implement → review loop. PR creation is handled by takt's `auto_pr` feature, not by workflow steps.

```
plan (opus) → implement (composer-2-fast) → review (codex) → COMPLETE
                                               ↓
                                              fix (composer-2-fast) → review
```

| Step | Persona | Edits | Policy | Description |
|------|---------|-------|--------|-------------|
| plan | planner | no | delivery-standards | Break requirements into ~500-line phases (Phase 0 = CI/CD) |
| implement | coder | yes | coding, delivery-standards | Build the current phase on a feature branch |
| review | reviewer | no | review, delivery-standards | Code review (BLOCKING/IMPORTANT only) |
| fix | coder | yes | coding, delivery-standards | Address review findings, then re-review |

### Delivery Standards Policy

Applied to all steps. Key rules:

- **Phase 0 = CI/CD setup** — GitHub Actions (lint + test) before any implementation
- **~500 lines per PR** — exceeding this is a planning defect
- **Independent phases** — each phase leaves the codebase in a working state
- **No draft PRs** — PRs are created ready-for-review
- **Tests required** — every phase includes tests for its code

## Setup

### 1. Install takt and clone this repo

```bash
npm install -g takt
git clone git@github.com:tomoya55/takt-config.git ~/takt-config
```

### 2. Link shared resources into `~/.takt/`

```bash
ln -s ~/takt-config/facets ~/.takt/facets
ln -s ~/takt-config/workflows ~/.takt/workflows
```

This makes all personas and workflow presets available globally. No per-project symlinks needed.

### 3. Configure `~/.takt/config.yaml`

Add `persona_providers` to your global takt config:

```yaml
persona_providers:
  planner:
    provider: claude
    model: opus
  coder:
    provider: cursor
    model: composer-2-fast
  reviewer:
    provider: codex
    model: codex
```

### 4. Disable GPG signing for takt worktrees

takt runs agents in isolated worktrees under `~/Work/src/github.com/<user>/takt-worktrees/`. If your git config requires GPG/SSH signing via 1Password (or similar), agents will fail to commit because they can't access the signing UI.

**Fix:** Use git's `includeIf` to disable signing only for takt worktrees.

Create `~/.gitconfig-takt`:

```ini
[commit]
	gpgsign = false
```

Append to `~/.gitconfig`:

```ini
[includeIf "gitdir:~/Work/src/github.com/<user>/takt-worktrees/"]
	path = ~/.gitconfig-takt
```

This disables commit signing inside takt worktrees while keeping it enabled everywhere else.

### 5. Per-project setup

Only project-specific files go in `.takt/`:

```
.takt/
├── tasks.yaml                           # task queue
├── tasks/
│   └── <slug>/
│       └── order.md                     # PRD
└── config.yaml                          # project overrides (optional)
```

### 6. Run

```bash
takt run
```

## Adding a New Preset

Create a new YAML file in `workflows/`:

```bash
cp workflows/phased-delivery.yaml workflows/quick-fix.yaml
# Edit steps as needed
git add workflows/quick-fix.yaml
git commit -m "feat: add quick-fix workflow preset"
git push
```

Available immediately via `~/.takt/workflows` symlink — no per-project linking needed.

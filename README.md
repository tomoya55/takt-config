# takt-config

Shared [takt](https://github.com/nrslib/takt) workflow presets and persona configurations for AI agent coordination.

## Structure

```
takt-config/
├── config.yaml               # Default provider/model and persona_providers
├── facets/
│   └── personas/
│       ├── planner.md         # Senior architect — plans phased milestones
│       ├── coder.md           # Full-stack developer — implements features
│       ├── reviewer.md        # Pragmatic reviewer — blocks on real issues only
│       └── integrator.md      # Release engineer — PR creation, CI checks, merging
└── workflows/
    └── phased-delivery.yaml   # Multi-phase delivery with per-phase PRs
```

## Persona → Provider Mapping

| Persona | Provider | Model | Role |
|---------|----------|-------|------|
| planner | Claude Code | opus | Planning and milestone definition |
| coder | Cursor | composer-2-fast | Implementation and fixes |
| reviewer | Codex | (default) | Code review (skips trivial findings) |
| integrator | Claude Code | sonnet | PR creation, CI evaluation, merging |

## Workflow Presets

### `phased-delivery`

A multi-phase delivery workflow where each milestone becomes a separate PR.

```
plan → implement → create-pr → evaluate → review → merge → [next phase or COMPLETE]
                                  ↑          |
                                  fix ←------┘
```

| Step | Persona | Edits | Description |
|------|---------|-------|-------------|
| plan | planner | no | Break requirements into ~500-line phases |
| implement | coder | yes | Build the current phase on a feature branch |
| create-pr | integrator | yes | Commit, push, and open a PR |
| evaluate | integrator | no | Check CI status, PR size, code quality |
| review | reviewer | no | Code review (BLOCKING/IMPORTANT only) |
| fix | coder | yes | Address review findings, then re-evaluate |
| merge | integrator | yes | Squash-merge, then loop to next phase or COMPLETE |

**Quality gates (evaluate step):**

- CI: all checks must pass
- PR size: <500 lines PASS, 500–999 WARNING, 1000+ BLOCKING
- Quality: only BLOCKING and IMPORTANT issues are flagged

## Usage in a New Project

### 1. Initialize takt

```bash
npm install -g takt
cd your-project
takt init
```

### 2. Clone this config repo (once per machine)

```bash
git clone git@github.com:tomoya55/takt-config.git ~/takt-config
```

### 3. Link shared resources

```bash
# Link personas (shared across all presets)
ln -s ~/takt-config/facets .takt/facets

# Link a workflow preset
ln -s ~/takt-config/workflows/phased-delivery.yaml .takt/workflows/phased-delivery.yaml
```

### 4. Add your project-specific files

```
.takt/
├── facets -> ~/takt-config/facets           # symlink
├── workflows/
│   └── phased-delivery.yaml -> ~/takt-config/...  # symlink
├── tasks/
│   └── your-prd.md                          # project-specific PRD
└── config.yaml                              # project-specific overrides (optional)
```

If you need to override the default provider/model for this project, create `.takt/config.yaml`:

```yaml
provider: claude
model: sonnet
language: ja
```

### 5. Run

```bash
takt run --workflow phased-delivery --task tasks/your-prd.md
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

Then link it in any project:

```bash
ln -s ~/takt-config/workflows/quick-fix.yaml .takt/workflows/quick-fix.yaml
```

# Org Conventions

Shared conventions that apply across all alphachu-volleyball repositories.

Each repo's `CLAUDE.md` builds on this document, adding or overriding repo-specific details.

---

## Versioning

- Follow **Semantic Versioning (semver)**: `MAJOR.MINOR.PATCH`
- Mark versions with Git tags (e.g., `v0.1.0`)
- Each repo is versioned independently
- Inter-repo dependencies are pinned to Git tags

## Branch Strategy

| Branch | Purpose | Merge Method |
|--------|---------|--------------|
| `main` | Stable release state | `release/*` → main: merge commit |
| `release/{version}` | Release integration | `feat/*`, `fix/*` → release: squash merge |
| `feat/*`, `fix/*` | Feature/bugfix work | Merged into release branch via PR |

Workflow: `feat/*` → `release/{version}` (squash) → `main` (merge commit) → tag

Documentation-only repos (e.g., `.github`) merge directly: `feat/*` → `main` (squash merge), without a release branch.

## Commit Convention

Follow the [Conventional Commits](https://www.conventionalcommits.org/) format.

```
<type>(<scope>): <subject>

feat(env): add random ball position mode
fix(wrapper): correct observation space shape
docs(readme): update architecture diagram
chore(ci): add ruff lint workflow
```

Primary types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`

## Python Repos

### Environment

- **Python**: 3.12+
- **Package management**: uv (`pyproject.toml` + `uv.lock`)
- Manage lock file with `uv lock`, install with `uv sync`

### Code Quality

- **Linter/Formatter**: ruff
- **Testing**: pytest
- ruff and pytest run automatically in CI

### Default ruff Config

Included in each repo's `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py312"
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]
```

## JavaScript/Web Repos

- Linter: eslint
- Package management: npm or yarn (decided per repo)

## CI/CD

### Python Repos (GitHub Actions)

| Trigger | Action |
|---------|--------|
| PR, push to main | ruff lint, pytest |
| tag push (`v*`) | Create release (if applicable) |

- Training is not run in CI (requires GPU, long-running)

### Web Repos (GitHub Actions)

| Trigger | Action |
|---------|--------|
| PR, push to main | eslint, build check |
| push to main | GitHub Pages deploy (champions) |

## Inter-Repo Dependencies

### pika-zoo → training-center

training-center references pika-zoo as a Git dependency, pinned to a tag:

```toml
[project]
dependencies = [
  "pika-zoo @ git+https://github.com/alphachu-volleyball/pika-zoo@v1.5.0",
]
```

When pika-zoo is updated, the training-center dependency must be updated accordingly.

### training-center → champions

- training-center exports ONNX models and publishes them to Hugging Face Hub
- champions fetches the model from Hugging Face Hub for inference
- External registries like PyPI are not used

## Artifact Management

### Model Files

- Do not commit directly to Git (tens to hundreds of MB)
- Publish ONNX models to Hugging Face Hub from training-center
- `models/checkpoints/` and `models/exported/` are included in `.gitignore`

### Game Assets (Sprites, Sounds)

- Files are small, so commit directly to Git

### Experiment Tracking

- Integrate with W&B or TensorBoard (SB3 native support)
- Used for tracking hyperparameters, reward curves, and ELO ratings

## Code Copying Policy

- No submodules — substantial customization is required
- When copying external code, always include:
  - Original source URL
  - License file (LICENSE)
  - Change log (ATTRIBUTION.md)

### Reference Sources

| Source | Copy Target | License |
|--------|-------------|---------|
| [helpingstar/pika-zoo](https://github.com/helpingstar/pika-zoo) | pika-zoo repo | MIT |
| [hankluo6/gym-pikachu-volleyball](https://github.com/hankluo6/gym-pikachu-volleyball) | pika-zoo repo (reference) | TBD |
| [gorisanson/pikachu-volleyball](https://github.com/gorisanson/pikachu-volleyball) | champions repo (fork) | UNLICENSED (TBD) |

## Hardware Reference

- AMD Ryzen 7 3700X (8C/16T), NVIDIA RTX 2080 Super (8GB)
- Low-dimensional vector observations + MLP policy — CPU (environment parallelization) is the bottleneck, not GPU
- Can run 8–16 parallel environments via SB3 `SubprocVecEnv`

## License

- pika-zoo: MIT License
- pikachu-volleyball (original): UNLICENSED (TBD)
- Original game assets: "(C) SACHI SOFT / SAWAYAKAN Programmers", "(C) Satoshi Takenouchi" 1997

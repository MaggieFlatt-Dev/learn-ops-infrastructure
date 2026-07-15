# Linting Tools Found

Only **ESLint** is configured. This is a Create React App project — no Python/pylint config exists (no `.pylintrc`, no `[tool.pylint]` in `pyproject.toml`, no `[pylint]` in `setup.cfg`).

## ESLint

| | |
|---|---|
| **Config location** | `package.json:39-41` — `"eslintConfig": { "extends": "react-app" }` |
| **Command** | `npx eslint ~/workspace/lms/learn-ops-client` |
| **Base config** | `eslint-config-react-app` (`node_modules/eslint-config-react-app/index.js`), which itself extends `./base` |

Since the project only sets `"extends": "react-app"` with no custom `rules` block, **all actual rule definitions come from the `eslint-config-react-app` package**, not from this repo's own config.

### Rule categories enforced (with examples)

- **Correctness / possible errors** — `no-undef` (error), `no-dupe-keys`, `no-unreachable`, `no-const-assign`, `use-isnan`, `valid-typeof` (all `warn`)
- **React Hooks correctness** — `react-hooks/rules-of-hooks` (**error**) — enforces hooks are only called at the top level; `react-hooks/exhaustive-deps` (`warn`) — flags missing dependencies in `useEffect`/`useCallback`/etc.
- **React-specific** — `react/jsx-no-duplicate-props`, `react/jsx-no-undef` (error), `react/no-direct-mutation-state`, `react/no-typos` (error)
- **Accessibility (jsx-a11y)** — `jsx-a11y/alt-text`, `jsx-a11y/anchor-is-valid`, `jsx-a11y/aria-role`, `jsx-a11y/iframe-has-title` (all `warn`)
- **Import hygiene** — `import/first` (error, imports must come before other code), `import/no-webpack-loader-syntax` (error), `import/no-anonymous-default-export` (`warn`)
- **Style/safety guards** — `eqeqeq` (`warn`, "smart" mode — allows `== null`), `no-eval`, `no-script-url`, `no-restricted-globals` (blocks confusing browser globals like `event`, `name`)

### Explicitly disabled rules

`node_modules/eslint-config-react-app/index.js:249-252`:
```js
// Disabled because of undesirable warnings
// See https://github.com/facebook/create-react-app/issues/5204 for
// blockers until its re-enabled
// 'react/no-deprecated': 'warn',
```
This rule, if enabled, would flag usage of deprecated React APIs (e.g. `componentWillMount`, `componentWillReceiveProps`, string refs, `ReactDOM.render` in newer React versions). It's commented out upstream in the `eslint-config-react-app` package itself (not something this repo disabled) due to false-positive noise tracked in that linked CRA issue.

## learn-ops-api

Only **pylint** is configured (no `.eslintrc*`, no `package.json` with `eslintConfig`, no `setup.cfg`/`pyproject.toml` pylint sections — the config lives entirely in `.pylintrc`). This is a pure Django/Python project (no `package.json`).

### pylint

| | |
|---|---|
| **Config location** | `.pylintrc` (repo root, 5 lines) |
| **Command** | `cd ~/workspace/lms/learn-ops-infrastructure && docker compose exec api pylint /app` — pylint has to run inside the `api` container (defined in `learn-ops-infrastructure/docker-compose.yml`); that container mounts the host `~/workspace/lms/learn-ops-api` directory to `/app`, so `/app` is the full in-container path to target. |

#### Rule categories enforced

`.pylintrc` doesn't add custom rules beyond a `good-names` allowlist (`i, j, ex, pk` — short variable names pylint would otherwise flag under `invalid-name`/C0103) and a `disable` list. Everything else runs on pylint's default rule set, which covers categories like:
- **Convention** (`C`) — e.g. `invalid-name`, `missing-module-docstring`
- **Refactor** (`R`) — e.g. `too-many-arguments`, `duplicate-code`
- **Warning** (`W`) — e.g. `unused-variable`, `broad-except`
- **Error** (`E`) — e.g. `no-member`, `undefined-variable`

#### Explicitly disabled rules

`.pylintrc:5` — `disable=broad-except,arguments-differ,missing-function-docstring`

- **`broad-except` (W0703)** — flags `except Exception:` or bare `except:` clauses. If enabled, it would catch every overly-broad exception handler in the Django codebase (e.g. views/services swallowing all exceptions instead of catching a specific type), which is exactly the kind of pattern that can silently mask real bugs since any error — not just the one you expected — gets caught and ignored.
- **`arguments-differ` (W0221)** — would flag method overrides (e.g. subclassed Django views/serializers) whose signature doesn't match the parent method's.
- **`missing-function-docstring` (C0116)** — would flag any function/method without a docstring.

---

## Skill vs. manual notes (`contribute-notes.md`)

**What matched:** The `eslintConfig: "react-app"` config and `no-eval` rule both lined up with the manual JS notes. On the Python side, the skill found the exact same three disabled rules (`broad-except`, `arguments-differ`, `missing-function-docstring`) and the same `good-names` allowlist (`i, j, ex, pk`) as the manual notes — and after refinement, the `broad-except` explanation matches almost word-for-word ("catching all exceptions can hide bugs").

**What it got wrong or missed on the first run:** The skill only reads static config, so it never runs the linters — it missed the actual violations the manual notes caught by hand (`no-unused-vars` in `LearnOps.js:8`, missing docstring + trailing newline in `LearningAPI/metrics.py`). The manual notes also mention `pylint-django` likely adding location/category info, but `.pylintrc` has no `load-plugins` entry, so the skill never surfaced that plugin. The first-run pylint command was also generic (`pylint <path>`) and didn't account for pylint needing to run inside the `api` container.

**What changed after refining the prompt:** Added the `docker compose exec api pylint <full path>` instruction, verified against `docker-compose.yml` (the `api` service mounts `learn-ops-api` to `/app`, so the real command is `docker compose exec api pylint /app`). Also added an explicit `broad-except` explanation to the skill instructions, so disabled-rule output now explains *why* a rule matters instead of just naming it, matching the manual notes' style.

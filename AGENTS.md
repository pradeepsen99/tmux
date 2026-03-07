# AGENTS.md

## Purpose
Guidance for coding agents working in this repository.
Focus areas: command workflows, single-test execution, and shell/tmux style conventions.

## Repository Layout
- `tmux.conf`: primary user configuration.
- `plugins/tpm/`: TPM scripts and tests.
- `plugins/tmux2k/`: tmux2k plugin/theme scripts.
- `plugins/tmux/`, `plugins/tmux-sensible/`: plugin dependencies.

## Source Of Truth And Scope
- Follow existing patterns in the file you touch.
- Treat `tmux.conf` as the top-level product in this repo.
- Treat plugin directories as upstream/vendor unless task explicitly targets them.
- Avoid broad refactors, mass reformatting, and unrelated cleanup.

## Cursor And Copilot Rules
No repository-specific rule files were found:
- `.cursorrules` not present
- `.cursor/rules/` not present
- `.github/copilot-instructions.md` not present
If these appear later, treat them as higher-priority instructions.

## Environment
- Shell: `bash`.
- Core tools: `tmux`, `git`, standard Unix utilities.
- Optional tooling: `shellcheck`, `shfmt`.
- tmux2k notes indicate bash 5.2+ is preferred for color rendering.

## Build, Lint, And Test
There is no compile/build step. Validate via tmux load checks, shell syntax checks, and TPM tests.

### Root Validation (from repo root)
- Parse/load config with isolated tmux server:
  - `tmux -f ./tmux.conf -L agents-check start-server`
  - `tmux -L agents-check kill-server`
- Reload config in a live session:
  - `tmux source-file ./tmux.conf`

### TPM CLI Workflows (from repo root)
- Install configured plugins: `./plugins/tpm/bin/install_plugins`
- Update all plugins: `./plugins/tpm/bin/update_plugins all`
- Update one plugin: `./plugins/tpm/bin/update_plugins tmux-sensible`
- Remove unconfigured plugins: `./plugins/tpm/bin/clean_plugins`

### Fast Lint Baseline
- Syntax check all TPM scripts: `bash -n plugins/tpm/**/*.sh`
- Syntax check all tmux2k scripts: `bash -n plugins/tmux2k/**/*.sh`
- Fallback when `globstar` is unavailable:
  - `find plugins/tpm -name '*.sh' -print0 | xargs -0 -n1 bash -n`
  - `find plugins/tmux2k -name '*.sh' -print0 | xargs -0 -n1 bash -n`

### Shellcheck (if installed)
- `shellcheck plugins/tpm/**/*.sh`
- `shellcheck plugins/tmux2k/**/*.sh`
- Targeted lint example:
  - `shellcheck plugins/tpm/scripts/install_plugins.sh`

### TPM Test Setup
TPM tests depend on `plugins/tpm/lib/tmux-test`.
If helpers are missing, initialize/setup first:
- `git -C plugins/tpm submodule update --init --recursive`
- `plugins/tpm/lib/tmux-test/setup`

### TPM Tests (especially single test)
Run from `plugins/tpm/`:
- All test files:
  - `for f in tests/test_*.sh; do bash "$f"; done`
- Single test file (preferred during iteration):
  - `bash tests/test_plugin_installation.sh`
  - `bash tests/test_plugin_update.sh`
  - `bash tests/test_plugin_clean.sh`
  - `bash tests/test_plugin_sourcing.sh`
Notes:
- Test files define `test_*` functions and call `run_tests`.
- `run_tests` comes from the tmux-test harness.

## Editing Guidelines

### Scope And Safety
- Edit only what is required for the task.
- Preserve existing tmux option names and plugin contracts.
- Do not change behavior outside the requested area.

### Imports / Sourcing
- Put path constants near the top (`CURRENT_DIR` or `current_dir`).
- Keep `source` statements near the top after constants.
- Quote source paths: `source "$HELPERS_DIR/file.sh"`.
- Prefer script-relative resolution via `dirname "${BASH_SOURCE[0]}"` patterns.

### Formatting
- Shebang for scripts: `#!/usr/bin/env bash`.
- Match local indentation style instead of normalizing globally:
  - TPM scripts commonly use tabs.
  - tmux2k scripts commonly use 4 spaces.
- Keep functions small and readable.
- Avoid trailing whitespace.

### Types And Data Handling (Shell)
- Shell has no static types: validate inputs defensively.
- Use `local` in functions.
- Quote expansions unless intentional splitting/globbing is required.
- Use arrays when list semantics already exist (plugin lists, parsed tokens).
- Preserve established parsing pipelines (`awk`, `sed`, `cut`) unless needed.

### Naming
- Constants/global-like vars: uppercase (`CURRENT_DIR`, `HELPERS_DIR`).
- Function/local vars: lowercase snake_case.
- Function names use verb-oriented snake_case (`install_plugin`, `get_tmux_option`).
- Tests use `test_*` naming.

### Error Handling
- Return non-zero on failure; emit concise stderr messages.
- Reuse existing helpers where present (`fail_helper`, `exit_value_helper`).
- Handle `git` and `tmux` failures explicitly.
- Do not silence errors unless file-local style already does.

### Control Flow
- Keep `main()` entrypoint pattern for executable scripts.
- Use guard clauses (early `return`/`continue`) for clarity.
- Favor explicit conditions over compact but opaque one-liners.

### Compatibility
- Keep compatibility with existing bash patterns in this repo.
- Avoid introducing new non-portable dependencies.
- Keep tmux option interfaces stable (`@plugin`, `@tmux2k-*`).

### Comments And Documentation
- Add comments only for non-obvious logic.
- Keep comments short and behavior-focused.
- Update docs/examples when command names or options change.

## Verification Checklist Before Finishing
- tmux config load check using isolated server.
- `bash -n` on edited scripts.
- `shellcheck` on edited scripts (if available).
- Run at least one relevant TPM single test when TPM behavior changes.

## Commit Hygiene
- Keep commits focused and minimal.
- Avoid unrelated vendor/plugin churn.
- Mention user-visible tmux behavior changes in commit messages.

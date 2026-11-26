# Repository rules (fork ulrichwi/thermiagenesis)

Supplements the user-level general rules and `CLAUDE.md` (loaded automatically; do not re-read it unless a specific section is needed).
Priority: safety > correctness > understanding of the repository > minimal changes > token efficiency.

## Origin and precedence

- This repo merges the work of David Aggeler (`david-aggeler/genesis-wp-ha`) into a fork. `CLAUDE.md` is his file: do not edit it here; put fork-specific rules in this file.
- Keep upstream-owned files mergeable: prefer additive changes, avoid reformatting or renaming files that come from upstream.
- Conflicts: the stricter rule wins, except language (see user-level rules: answers in German, code/identifiers/comments in English).
- Not available in this environment, do not attempt to use them: `mcp__gold__*` / `mcp__copper__*` (openbrain), `openbrain-client`, `SendMessage`/`ListAgents` peers, `/container/...` paths, zsh syntax. Where `CLAUDE.md` requires gold records (issue, change request, issue_fix, requirement, ac), the GitHub equivalents apply in this fork, see "Change tracking".

## Change tracking (replaces the gold record workflow of `CLAUDE.md`)

- Every repo change is tracked by a GitHub issue in `ulrichwi/thermiagenesis`: defect/gap/feature with the problem, the intended change, and acceptance criteria as a checklist. Search existing issues first. Docs-only and agent-instruction edits need no issue.
- Implement on a branch `issue-<n>-<short-name>`, test-first against the emulator, gate green (see "Windows environment").
- Before opening the PR: check the diff against every acceptance criterion and write down the evidence. A green gate alone is not proof.
- The PR targets `main` (the fork's trunk; `master` is the obsolete upstream state) and references the issue (`Closes #<n>`). Creating issues, branches, PRs, comments and merges on GitHub needs explicit user approval each time, except local commits on the branch when requested. Never open issues, PRs or comments against `CJNE/thermiagenesis` or `david-aggeler/genesis-wp-ha`.
- Library fixes (registers, decoding, transport, emulator) belong to the `genesis-wp` library, not to this repo. Report them as an issue text for the user; the integration here only bumps the pinned SHA.

## Hard safety rules (never relaxed by any other rule)

- The real heat pump is read-only. Never send Modbus function codes 5, 6, 15, 16; never contact the real device, not even for reads. Tests use only the loopback emulator.
- Writable entities exist only behind the explicit opt-in and are tested only against `genesis-wp-emulator`.
- The integration never decodes registers and never imports pymodbus; the register map belongs to the `genesis-wp` library.
- Never commit: host names, device serial numbers, manufacturer documents, `openbrain-client.local.yaml`, `.logs/`, `scratchpad/`, secrets.
- No push, force-push or push to `upstream`/`cjne` without explicit approval, which overrides the "push after every commit" rule in `CLAUDE.md`. Commits only on request, on a branch, with the `Co-authored-by` trailer.

## Languages and required knowledge (scope for analysis)

Relevant:
- Python >= 3.14 (asyncio, type hints): Home Assistant custom integration patterns (config entries/flows, `DataUpdateCoordinator`, entity descriptions, options, diagnostics, translations).
- pytest with `pytest-homeassistant-custom-component`, `pytest-socket` (loopback only), emulator fixtures in `tests/harness.py`.
- JavaScript (ES modules, no build step) for Lovelace cards and the custom panel in `custom_components/genesis_wp/frontend/`; Node `.mjs` tests in `tests/frontend/`.
- JSON (`manifest.json`, `strings.json`, `translations/`, `hacs.json`), YAML (GitHub workflows, `openbrain-client.yaml`), small bash scripts in `scripts/`.
- Modbus/WPM-G only at concept level (what a register, scale, sentinel is); details come from the library.

Not relevant: other languages and frameworks, build toolchains for the frontend (there is none), anything outside Home Assistant custom integrations.

## Scope of search and reading

- Exclude from searches: `.venv/`, `worktrees/`, `scratchpad/`, `__pycache__/`, `.git/`, `.claude/skills/` (except `publish/`), `.claude/agents/`.
- Integration code: `custom_components/genesis_wp/`; tests: `tests/`; gate and merge tooling: `scripts/`, `openbrain-client.yaml`. Start there instead of scanning the whole tree.
- Read whole files when they are directly affected (including callers, callees, tests); use `view_range` or grep for large unrelated files such as translations and frontend bundles.
- Do not read `LICENSE`, `VERSION`, `.github/labels.yml`, release-drafter or dependabot files unless the task concerns them.

## Windows environment (host of this clone)

- Shell is PowerShell. Python 3.14 is installed but not on `PATH`: use `py -3.14`. The `.venv` exists (gitignored); recreate it with `py -3.14 -m venv .venv`, then `.venv\Scripts\python.exe -m pip install -r requirements_test.txt -r requirements_dev.txt`. Node is provided by `nodejs-wheel-binaries` in `requirements_test.txt`; no separate install is needed for the `.mjs` frontend tests.
- **`pytest` does not run natively on Windows:** `homeassistant` imports the Unix-only module `fcntl` (`ModuleNotFoundError`). Do not stub `fcntl` or otherwise patch the environment to force a run. Only the `static` rung runs natively; `unit` and `integration` run in WSL2 (next bullet). Docker is not installed.
- **WSL2 test environment:** distro `Ubuntu` (Python 3.14, `uv` in `~/.local/bin`), clone at `~/thermiagenesis` with its own `.venv`, created via `uv venv --python 3.14 .venv` and `uv pip install --python .venv/bin/python -r requirements_test.txt -r requirements_dev.txt`. The clone only sees committed work: sync with `git pull /mnt/c/Users/ulric/src/repos/thermiagenesis main` (the Windows clone is its `origin`; uncommitted Windows changes are not tested). No `sudo` is needed, and the agent has no sudo password.
- Run the gate in WSL, capture before filtering, and report the real exit code: `wsl -d Ubuntu -- bash ~/wsl_gate.sh`, where the script runs `.venv/bin/python -m compileall -q custom_components` and `.venv/bin/python -m pytest -q -p no:cacheprovider > scratchpad/gate.out 2>&1; echo "pytest rc=$?"`. The full suite (278 tests, last verified at commit `1fc6567`) takes about 20 minutes: run it async. GitHub Actions runs it with `-n auto` from a repo `.venv` (about 13 minutes; green at `12ed5d8`). Write helper scripts under `scratchpad\` with the `create` tool and copy them into WSL with `tr -d '\r'` (CRLF breaks bash); avoid nested quotes in `wsl ... bash -c`.
- Gate commands in `CLAUDE.md` use zsh and `.venv/bin/python`; inside WSL they apply as written (bash instead of zsh: use `rc=$?`). On Windows only `.venv\Scripts\python.exe -m compileall -q custom_components` is meaningful.
- Gate rungs by intent: `static` = `python -m compileall custom_components`; `unit` = `python -m pytest` (hermetic); `integration` = pytest including emulator tests. The `system` rung (real plant) is never run from here.
- `AGENTS.md` is a plain 9-byte text file (`CLAUDE.md`) on Windows, not a symlink (`core.symlinks=false`). Do not rewrite it; it is correct on Linux/macOS.

## Manual check in a throwaway Home Assistant (emulator only)

Only the integration's `emulated` mode is used: never choose `real` in the config flow and never point it at the plant. Purpose: look at entities, panel and cards without a plant. Everything lives in WSL under `~/ha-test`, outside the repo.

- **Setup:** own venv (`uv venv --python 3.14 --seed venv`), then `uv pip install --python venv/bin/python "homeassistant==2026.10.0" "genesis-wp[emulator] @ git+https://github.com/david-aggeler/genesis-wp@<SHA from manifest.json>"`. Config dir `config/` with `custom_components/genesis_wp` symlinked to `~/thermiagenesis/custom_components/genesis_wp` (sync that clone first, see above). Never install into the repo `.venv`: HA installs packages at runtime.
- **`configuration.yaml`:** `homeassistant`, `http`, `frontend`, `config`, `onboarding`, `history`, `recorder`, `logger` (debug for `custom_components.genesis_wp` and `genesis_wp`). Not `default_config`, it pulls in discovery and Bluetooth.
- **Loopback only:** HA 2026.10 keeps HTTP settings in `config/.storage/http`. `http.server_host` from YAML is only `pending` and reverts to `stable` after 5 minutes (HA then exits with code 100 and listens on `0.0.0.0`). While HA is stopped, set `data.stable.server_host` to `["127.0.0.1"]` and keep `pending` as `null` (removing the key breaks the http setup). Verify with `ss -ltn | grep 8123`.
- **Missing compiler:** `pymicro-vad` and `pyspeex-noise` cannot be built without g++ (no sudo), and HA's service loader imports `assist_satellite`, so the UI hangs on "Loading…". Install prebuilt wheels into the test venv: `uv pip install --python venv/bin/python --only-binary :all: pymicro-vad pyspeex-noise`.
- **Run and stop:** start `venv/bin/hass -c config` as the foreground command of an async `powershell` call (`wsl -d Ubuntu -- bash <script>`), not detached. Stop it with `stop_powershell`, then confirm with `pgrep -f venv/bin/hass`; kill leftovers by PID only. Never run two instances on the same config.
- **Onboarding and flow over REST** (typing into the login form is unreliable in the browser tool): `POST /api/onboarding/users`, `/auth/token`, `/api/onboarding/{core_config,analytics,integration}`, then `/api/config/config_entries/flow` with handler `genesis_wp`, `next_step_id=emulated`, `profile=<profile>`, and the module booleans. For the browser, run `/auth/login_flow` with `redirect_uri=http://localhost:8123<path>?auth_callback=1` and open `<redirect_uri>&code=<code>&state=<base64 of {"hassUrl":"http://localhost:8123","clientId":"http://localhost:8123/"}>`. The panel is at `/genesis-wp`.
- **Credentials:** random and throwaway, stored only in `~/ha-test/credentials.txt` in WSL; never copy them into the repo or committed files.
- **Checks:** entity states via `GET /api/states`; no `ERROR` lines from `genesis` in `~/ha-test/ha.log`; no write-related lines; `ss -tnp` shows only loopback sockets for the hass process.
- **Known defect (this repo, not the library):** at emulator start and in the emulated config flow, HA warns about blocking `read_text`/`open`/`scandir` calls. HA attributes them to `custom_components/genesis_wp/__init__.py` (`_start_emulator`, `get_profile`) and `config_flow.py` (`async_step_emulated`: `get_profile`, `packaged_profiles`). The library functions are synchronous by design; the integration must call them via `hass.async_add_executor_job`.

## Code conventions

- Follow existing patterns and keep the existing style of each file: the code base is not black-formatted (black would rewrite 40 of 43 files), so never run black or another formatter over existing files and avoid mass reformatting. `flake8` (config in `setup.cfg`, CI job `Flake8`) must stay clean; keep imports in the `reorder-python-imports` style (one import per line).
- A not-fitted or unavailable value (`None` from the library) results in an unavailable/unknown entity, never in `0` or a coerced number.
- No blocking I/O in the event loop. Log meaningful events (no feedback loop without logs), without host names or serial numbers.
- Entity names and strings follow Home Assistant conventions; German text only in `translations/de.json`. Use US English spelling in code, comments, and commit messages.
- `manifest.json` pins `genesis-wp` to a full commit SHA, and `requirements_test.txt` must pin the same SHA (`tests/test_manifest.py` checks this). Pin bumps need a reason and the library's license check stays MIT-compatible.
- Never weaken `tests/conftest.py` (network guard) or the read-only tests (`test_read_only.py`, `test_network_guard.py`).

## Token efficiency (secondary)

- Answer concisely; no recap of unchanged work; link files instead of quoting them.
- Parallelize independent reads and searches; do not repeat known results.
- Pipe long test output into `scratchpad\*.out` and inspect it selectively, but always report the real exit code.

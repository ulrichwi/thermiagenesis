---
applyTo: "tests/**"
---

# Tests

- Hermetic: loopback only, no real device, no secrets. Never weaken `tests/conftest.py` (network guard) or the read-only tests (`test_read_only.py`, `test_network_guard.py`).
- Emulator tests use the `start_emulator` fixture; firmware profiles and constants are in `tests/harness.py`, shared setup in `tests/helpers.py`. Cover every shipped firmware profile when behavior depends on it.
- Writable entities are tested only with the write opt-in enabled and only against the emulator.
- Fixtures and firmware profiles belong to the library; do not copy or edit them here.
- Add a test only for behavior that is introduced or fixed; prefer existing test files. Never skip, delete or loosen a test to get green.
- Frontend checks run through `tests/node_harness.py` (`run_node_harness`) with scripts in `tests/frontend/*.mjs`. The harness must end its stdout with `HARNESS-COMPLETE`. Node comes from `nodejs-wheel-binaries` in `requirements_test.txt`.
- Run the narrowest covering test first (`python -m pytest tests\test_x.py`), then the full suite before reporting. Capture the output in `scratchpad\` and report the real exit code.

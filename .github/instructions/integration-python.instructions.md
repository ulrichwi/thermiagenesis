---
applyTo: "custom_components/**/*.py"
---

# Integration code (Python)

- Read-only against the plant: use only the `genesis-wp` client API. No Modbus write calls, no pymodbus imports, no register decoding here.
- A `None` value from the library means not fitted or unavailable: the entity is not created or is unavailable/unknown, never `0`.
- Async only; no blocking I/O in the event loop. Use `DataUpdateCoordinator`, config entries and entity descriptions as in the existing modules.
- Register-derived names, scales, units and enums come from the library. Do not duplicate them in `const.py`; map library values to entities.
- Writable entities (switch, number, select, climate setpoints) only behind the explicit opt-in, default off.
- User-visible text goes into `strings.json` and `translations/en.json`; German only in `translations/de.json`. Keep `strings.json` and `translations/en.json` in sync.
- Log relevant state changes and failures with the standard `logging` module; never log host names or serial numbers.
- Keep existing `DHF-REQ` comments (traceability to the original author's records); do not invent new ones.
- After changing the library pin in `manifest.json`, `requirements_test.txt` must carry the same SHA (`tests/test_manifest.py`).

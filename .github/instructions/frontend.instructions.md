---
applyTo: "custom_components/genesis_wp/frontend/**"
---

# Frontend (Lovelace cards and sidebar panel)

- Plain JavaScript ES modules, no build step, no bundler, no new npm dependencies.
- The frontend works only with Home Assistant state and the HA websocket (`hass`, `callWS`). It never talks to the heat pump directly.
- Short register labels live in `labels.js`: English is the fallback for every language; add languages there instead of hard-coding text in cards.
- Unknown or unavailable states are shown as such (HA `localize`) or not drawn; never substitute `0` or another value.
- Every behavior change gets a check in `tests/frontend/*.mjs`, driven from the matching pytest file (`test_card.py`, `test_device_card.py`, `test_overview_card.py`, `test_panel.py`, `test_labels.py`).
- Format with prettier as configured in `.pre-commit-config.yaml`.

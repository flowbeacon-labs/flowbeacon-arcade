# AGENTS.md — Flowbeacon Arcade Shared Rules

This repo is the central portal for independently hosted Flowbeacon browser games.

**Mandatory quality policy:** [docs/GAME_QUALITY_STANDARD.md](docs/GAME_QUALITY_STANDARD.md). Read and apply it before coding, testing, reviewing or publishing any arcade-related game. Keep this policy aligned with each separate game's root AGENTS.md.

## Nonnegotiable requirements

- iPhone Safari hold/touch controls (movement, joystick, fire, throttle, buttons) must not trigger long-press context menus, text selection, previews or scrolling. Use **scoped** `-webkit-touch-callout:none`, `-webkit-user-select:none`, appropriate `touch-action`, Pointer Events and `contextmenu`/ `selectstart` suppression on *control surfaces only*. Never globally disable browser/native form behavior.
- Handle `pointerup`, `pointercancel`, `lostpointercapture`, window blur and multi-touch so no input gets stuck.
- Keep HUD and controls usable without overlap on iPhone portrait/landscape and desktop at widths 375/590/768/1280.
- Do not allow a new feature to break game startup, links, missions, saves, audio, camera or previously working controls.
- For WebGL, verify in a **real browser**, not only a mocked WebGL context; report compile/link errors visibly.
- Icon buttons must have accessible names, finger-sized targets, and state changes must not replace/remove their icons.
- Never state “works on iPhone” without a physical iPhone Safari test. Label device testing as pending when unavailable.
- Confirm CI and GitHub Pages deployment; do not equate commits with verified publishing.

## Before finishing a task

Read the central policy, inspect the target game's existing AGENTS.md and tests, add a regression test for the reported defect, run tests/actual browser launch, check responsive overlap, and report measured passes, failures, limitations and deployment state.

Games live in different GitHub repos: changes to this AGENTS.md do **not** propagate automatically. Update each target repository's AGENTS.md when standards change.

## Helpful references

- Shared standard: docs/GAME_QUALITY_STANDARD.md
- ChatGPT Project instructions: docs/CHATGPT_PROJECT_INSTRUCTIONS.md
- Main arcade: https://flowbeacon-labs.github.io/flowbeacon-arcade/

# Flowbeacon Arcade — Shared Mobile Game Quality Standard

**Applies to:** all Flowbeacon browser games and shared arcade UI, especially iPhone Safari, desktop Chrome/Edge and Android Chrome.

**Goal:** stop repeating defects in controls, startup, navigation and layout. This is a **required release gate**, not merely an optional design suggestion.

## 1. iPhone touch contract — mandatory

All controls that users may press or hold (joystick, D-pad, throttle, steering, jump, fire, pause, camera, sound, fullscreen, game actions and icon buttons) MUST:

1. Respond to a real touch or pointer event (do not depend solely on hover, double-click, keyboard or mouse).
2. Never open Safari's text-selection handles, long-press copy/share callout, link preview or browser context menu while held.
3. Not cause page scrolling, pull-to-refresh, unwanted pinch zoom, or a stuck control while performing gameplay gestures. Keep ordinary page/accessibility zoom and form interactions usable outside gameplay.
4. Release input state on `pointerup`, `pointercancel`, `lostpointercapture`, and when the tab/window loses focus. Two simultaneous touches must not cancel each other.
5. Distinguish hold actions (throttle/move/fire) from single-tap toggles (brake/pause/settings). Holding must not trigger repeated unintended toggles.
6. Work with 1-finger and multi-touch on a physical iPhone, including switching portrait/landscape and resuming from an interruption.
7. Remain reachable with finger-sized hit areas (target >=44x44 CSS px where practical). Icon-only controls need descriptive `aria-label`, visible focus state, and a tooltip/title or on-screen Help legend.

**Scoped CSS baseline** (adjust selectors to each game; do not apply globally to body/menu/forms):

```css
.game-control,
.game-control *,
.game-joystick,
.game-joystick * {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}
.game-control--hold,
.game-joystick {
  touch-action: none;
  overscroll-behavior: contain;
}
.game-control--tap {
  touch-action: manipulation;
}
```

**Scoped JS baseline:** use Pointer Events, capture when applicable, `preventDefault()` on game-control gestures when cancelable, and prevent `contextmenu` / `selectstart` **only** for actual game-control surfaces. Never globally suppress text selection, touch events or native behavior on the entire document. Do not use a time-delay workaround or rely on CSS alone. Set and clear held-state robustly, including cancellation and blur; keep mouse, keyboard, VoiceOver and focus behavior intact.

### Example pointer-input pattern

```js
function bindHoldControl(element, onStart, onStop) {
  const pointers = new Set();
  const stop = e => {
    if (!pointers.has(e.pointerId)) return;
    pointers.delete(e.pointerId);
    if (element.hasPointerCapture?.(e.pointerId))
      element.releasePointerCapture(e.pointerId);
    if (pointers.size === 0) onStop();
  };
  element.addEventListener('pointerdown', e => {
    if (e.pointerType === 'mouse' && e.button !== 0) return;
    e.preventDefault();
    pointers.add(e.pointerId);
    element.setPointerCapture?.(e.pointerId);
    if (pointers.size === 1) onStart();
  });
  ['pointerup', 'pointercancel'].forEach(type => element.addEventListener(type, stop));
  element.addEventListener('lostpointercapture', stop);
  element.addEventListener('contextmenu', e => e.preventDefault());
  element.addEventListener('selectstart', e => e.preventDefault());
  // The game should also call onStop() for active controls on window blur/visibilitychange.
}
```

This pattern is a reference: review it against each game's input model; for example, `touch-action: none` is appropriate for joystick/hold surfaces but not for every tap button.

## 2. Safe UI and responsive layout

- Test phone landscape AND portrait, narrow desktop, tablet and ordinary desktop (minimum widths: 375, 590, 768, 1280 CSS px).
- Nothing important may overlap the playfield or controls. HUDs are compact, hideable, and respect `env(safe-area-inset-*)`; avoid large mission/status panels over gameplay.
- Do not hide critical actions behind hover or long text labels. Keep icon controls understandable and accessible.
- Never overwrite an icon button's `innerHTML` or `textContent` when changing state. Update a label/state element or `aria-pressed` instead.
- Leave a visible way back to selection/settings without leaving the overall game; pause/resume must not corrupt the session.
- Show useful start, loading, unsupported WebGL and runtime errors instead of a blank screen or inert Start button.

## 3. WebGL/canvas and game lifecycle

- Check shader compile AND link status; uniforms shared between stages must have compatible types/precision.
- Check camera matrices are finite; verify the player and ground are actually inside the active camera frustum.
- Test launch with the *real browser GPU/WebGL path*, not just mocked APIs.
- Avoid allocating GPU buffers/textures every frame; free replaced resources and handle WebGL context loss.
- Reset correctly after crash/restart, and keep mission/score/save migration behavior stable.
- Clamp elapsed time; pause if the app is backgrounded; ensure touch controls don't remain held.
- Do not assume GitHub Pages deployed solely because a code commit succeeded.

## 4. Required QA before claiming an update is complete

**Static checks:** JavaScript/HTML/CSS parse; controls exist; links resolve; no accidental third-party assets or missing files.

**Automated interaction checks:**
- Every gameplay control receives pointerdown and releases on up/cancel/lostcapture/blur.
- Dispatch a cancelable `contextmenu` event on each *holdable control* and confirm it is prevented.
- Confirm `-webkit-touch-callout:none`, `user-select:none`, and appropriate `touch-action` on each hold surface.
- Confirm controls work as designed, no double toggle, and icon/ARIA state survives repeated clicks.
- Launch, start, play an action, pause, resume, restart, return to selection; verify no runtime console errors.
- Check bounding boxes of HUD + controls at 375/590/768/1280px; ensure no overlap or off-screen controls.
- Check persisted state remains intact and local-only saves can be exported/recovered where applicable.
- For WebGL: real Chromium startup, shader-link check, aircraft/ground visible and camera reset.

**Device QA:**
- Test on a **real iPhone Safari** before declaring iPhone behavior fixed. Synthetic desktop WebKit/Chromium tests alone do **not** establish iOS Safari behavior.
- Hold each touch control >1.5 seconds and verify no selection/callout; use simultaneous joystick + throttle; background and resume.
- Check portrait/landscape, notch/safe areas, audio after gesture, and full-screen/scroll behavior.

If no physical iPhone is available, **report iPhone QA as pending** and do not say "fully tested" or "fixed on iPhone." A passing mock test is not proof of iOS behavior.

## 5. Standard development and delivery workflow

1. Inspect `AGENTS.md`, this standard, README, current implementation, existing tests and user-reported failures.
2. Write a compact risk list and decide acceptance criteria **before** modifying gameplay.
3. Implement minimum reversible change and preserve previously working behavior.
4. Add a regression test for the exact reported defect and related interactions.
5. Run project tests and actual browser flow; attach screenshots/logs when meaningful.
6. Verify GitHub Actions, GitHub Pages deployment state and URL with cache-busting when needed.
7. Report: changed files, test counts, browser/device coverage, remaining risks and the working link.

**Never** claim tests, deployment, device validation or release readiness without actual evidence. Do not add new gameplay features when a launch or input regression is unresolved.

## 6. Review checklist

- [ ] No iOS long-press/callout/text selection on controls
- [ ] Joystick/hold state resets on cancel, capture loss and blur
- [ ] Multitouch (move + action) works
- [ ] Controls fit safe areas and do not obstruct view
- [ ] All icon buttons are labeled, not overwritten on toggle
- [ ] Start, pause, resume, restart and back navigation work
- [ ] No browser-console or WebGL shader errors
- [ ] Screens/controls checked at common mobile/desktop widths
- [ ] Unit/regression and browser smoke tests pass
- [ ] Real iPhone Safari test status documented
- [ ] Existing functionality and saved data preserved
- [ ] GitHub Pages deployment confirmed and actual URL supplied

## 7. Scope and maintenance

Each game is in a **separate GitHub repository**. A file in the arcade hub is **not inherited** automatically by other repositories. Every game repository therefore has its own root `AGENTS.md` copy of the essential rules, pointing here as the canonical policy. When updating this standard, update those local files or an automated synchronizer.

Canonical URL: https://github.com/flowbeacon-labs/flowbeacon-arcade/blob/main/docs/GAME_QUALITY_STANDARD.md

**Important:** this document sets quality expectations; it does not retroactively change existing games or make a test pass by itself. Existing games require a separate code audit/patch and actual validation.

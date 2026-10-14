# Product Requirements Document: Dark Theme

## 1. Overview

Add a dark theme option to the existing browser-based Memory Game in `memory-game.html`. Players should be able to switch between light and dark appearances without changing gameplay, losing saved scores, or interrupting an active game.

This document captures the requirements and regression criteria for the dark-theme feature in `memory-game.html`.

## 2. Product Goals

- Offer a comfortable dark appearance for playing in low-light environments.
- Make switching themes simple and immediately visible.
- Preserve all existing game behavior and the current light appearance.
- Respect accessibility needs, including readable contrast, visible keyboard focus, and reduced-motion preferences where relevant.

## 3. Users and Use Cases

- A player prefers a dark interface and wants to enable it before or during a game.
- A player wants the game to remember their theme choice on a later visit.
- A player has an operating-system dark preference and expects a sensible initial theme when they have not made an in-game choice.
- A player uses keyboard controls or assistive technology and needs the theme control and game to remain accessible.

## 4. Scope

### In scope

- Add a clearly labeled theme toggle or equivalent accessible control to the game interface.
- Provide light and dark color themes for the page, controls and statistics, grid-size buttons, cards, best-scores area, and win dialog.
- Apply a suitable initial theme based on the operating-system preference unless the player has previously selected a theme.
- Persist an explicit player selection in browser storage, independently of saved game scores.
- Apply theme changes immediately without reloading or resetting the current game.

### Out of scope

- Changes to card-matching rules, scoring, ratings, timer behavior, grid sizes, or animations.
- New game modes, themes beyond light and dark, or external libraries and services.
- Changes to the existing best-score format or the `memoryGameScores` local-storage key.

## 5. Functional Requirements

1. The interface must provide an accessible control for switching between Light and Dark themes.
2. The control must be operable by mouse, touch, and keyboard, and expose its current state with an appropriate accessible name and state.
3. A theme change must update the full game interface immediately, including any currently visible win dialog, without changing game state.
4. If the player has not saved a theme choice, the initial theme must follow the browser's `prefers-color-scheme` setting. If that preference is unavailable, Light is the fallback.
5. Once the player selects a theme, that choice must persist across reloads and take precedence over the operating-system preference.
6. Theme preference storage must use a separate key from `memoryGameScores`. Existing scores must remain readable and unchanged.
7. Theme colors should be defined centrally (for example, with CSS custom properties) so all relevant surfaces use the active theme consistently.
8. Card states must remain distinguishable in both themes: face-down, face-up, matched, and keyboard-focused.
9. Existing game controls, status text, cards, and dialog must retain their current functionality and semantic accessibility attributes.

## 6. Non-Functional Requirements

- Keep the game self-contained in its existing HTML file and introduce no external runtime dependencies.
- Maintain the current responsive layout on mobile and desktop.
- Meet WCAG AA contrast targets for normal text (4.5:1) and large text (3:1) in both themes. Non-text controls and focus indicators must remain clearly perceivable.
- Theme changes must not cause a page reload, a layout shift, or a visible flash of the wrong theme where practical.
- Preserve the current light theme appearance as closely as possible.

## 7. Acceptance Criteria

- [ ] A player can switch between Light and Dark using the visible theme control.
- [ ] The selected theme is applied to the page background, text, controls, game cards, best-scores section, and win dialog.
- [ ] Changing the theme during a round preserves the board, flipped and matched cards, move count, timer, rating, and selected grid size.
- [ ] Changing the theme while the win dialog is open restyles the dialog without closing it or changing the final statistics.
- [ ] With no saved choice, the initial theme follows the operating-system preference; Light is used when no preference is available.
- [ ] After a manual selection, the selected theme remains active after reloading, even if the operating-system preference differs.
- [ ] Existing best scores remain intact after switching themes and after reloading.
- [ ] Grid-size selection, card matching, keyboard card activation, timer, rating, win flow, Play Again, and score display continue to work in both themes.
- [ ] Keyboard focus is visible and text and controls remain legible in both themes.
- [ ] The layout remains usable at narrow and wide viewport sizes.

## 8. Regression Verification

Before and after implementation, verify the following in a browser:

- Start and complete a round on both 4x4 and 6x6 grids.
- Flip cards using both mouse/touch and keyboard; confirm matches and non-matches still behave correctly.
- Confirm moves, timer, rating, win dialog, Play Again, and best-score updates work as before.
- Switch themes during an active round and while the win dialog is open; confirm no game state resets.
- Reload with each theme selected and confirm preference persistence and score preservation.
- Check the initial theme with both light and dark operating-system preferences and with no stored selection.
- Check contrast, focus visibility, and responsive layout in both themes.

## 9. Implementation Notes

The current stylesheet already uses CSS custom properties for several colors. Extend that approach into complete light and dark theme token sets rather than adding isolated dark-mode overrides. Keep theme selection separate from game-state logic and keep preference storage separate from score storage. Do not alter the current light-theme default appearance beyond what is necessary for the theme control.

## 10. Success Measure

The feature is successful when players can choose and retain either theme, all game surfaces remain clear and accessible, and the regression checklist passes without changes to gameplay or existing saved scores.

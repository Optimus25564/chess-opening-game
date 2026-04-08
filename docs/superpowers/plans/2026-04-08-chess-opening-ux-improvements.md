# Chess Opening UX Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Improve the chess opening learning game with SVG pieces, error feedback with consequence demos, auto-play opponent, and move navigation controls.

**Architecture:** Single-file HTML app (`index.html`). All changes go into this one file — CSS additions in the `<style>` block, data additions in the `OPENINGS` array, and JS changes in the `<script>` block. No external dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, inline SVG

**Spec:** `docs/superpowers/specs/2026-04-08-chess-opening-ux-improvements-design.md`

**Security note:** This is a local-only learning tool with no user-generated content. All data rendered via innerHTML is from hardcoded OPENINGS data and internal state — no external or user input is involved. innerHTML usage is safe in this context.

---

### Task 1: Replace Unicode pieces with inline SVG

**Files:**
- Modify: `index.html:869-872` (replace `PIECE_SYMBOLS` with `PIECE_SVG`)
- Modify: `index.html:902-932` (update `renderBoard()` to use SVG)
- Modify: `index.html:7-8` (add `.square svg` CSS)

This is the highest-impact visual change. The current Unicode characters render white pieces as outlines that are nearly invisible on light squares. SVG pieces with proper fill+stroke solve this completely.

- [ ] **Step 1: Add PIECE_SVG data object**

Replace the `PIECE_SYMBOLS` object at line 869-872 with a `PIECE_SVG` object containing SVG path data for all 12 pieces (6 white + 6 black). Use the Colin M.L. Burnett chess piece design (public domain, same as lichess).

In `index.html`, replace:
```js
const PIECE_SYMBOLS = {
  K: '♔', Q: '♕', R: '♖', B: '♗', N: '♘', P: '♙',
  k: '♚', q: '♛', r: '♜', b: '♝', n: '♞', p: '♟'
};
```

With the full `PIECE_SVG` object. Each entry is a function that returns an SVG string. White pieces: `fill:#fff, stroke:#333, stroke-width:1.5`. Black pieces: `fill:#333, stroke:#333, stroke-width:1.5` with white stroke for detail lines.

```js
// SVG piece data — Colin M.L. Burnett design (public domain)
function pieceSVG(paths, isWhite) {
  const fill = isWhite ? '#fff' : '#333';
  const stroke = '#333';
  const detailStroke = isWhite ? '#333' : '#fff';
  return `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45" style="width:100%;height:100%">${paths(fill, stroke, detailStroke)}</svg>`;
}

const PIECE_SVG = {
  K: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M22.5 11.63V6M20 8h5"/>
      <path d="M22.5 25s4.5-7.5 3-10.5c0 0-1-2.5-3-2.5s-3 2.5-3 2.5c-1.5 3 3 10.5 3 10.5"/>
      <path d="M12.5 37c5.5 3.5 14.5 3.5 20 0v-7s9-4.5 6-10.5c-4-6.5-13.5-3.5-16 4V27v-3.5c-3.5-7.5-13-10.5-16-4-3 6 5 10 5 10V37z"/>
      <path d="M12.5 30c5.5-3 14.5-3 20 0M12.5 33.5c5.5-3 14.5-3 20 0M12.5 37c5.5-3 14.5-3 20 0" stroke="${d}"/>
    </g>`, true),
  Q: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M9 26c8.5-1.5 21-1.5 27 0l2.5-12.5L31 25l-3.5-12.5-5.5 8-5.5-8L13 25 5.5 13.5 9 26z"/>
      <path d="M9 26c0 2 1.5 2 2.5 4 1 1.5 1 1 .5 3.5-1.5 1-1.5 2.5-1.5 2.5-1.5 1.5.5 2.5.5 2.5 6.5 1 16.5 1 23 0 0 0 1.5-1 0-2.5 0 0 .5-1.5-1-2.5-.5-2.5-.5-2 .5-3.5 1-2 2.5-2 2.5-4-8.5-1.5-18.5-1.5-27 0z"/>
      <path d="M11.5 30c3.5-1 18.5-1 22 0M12 33.5c6-1 15-1 21 0" stroke="${d}"/>
      <circle cx="6" cy="12" r="2"/><circle cx="14" cy="9" r="2"/><circle cx="22.5" cy="8" r="2"/><circle cx="31" cy="9" r="2"/><circle cx="39" cy="12" r="2"/>
    </g>`, true),
  R: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M9 39h27v-3H9v3zM12.5 32l1.5-2.5h17l1.5 2.5h-20zM12 36v-4h21v4H12z"/>
      <path d="M14 29.5v-13h17v13H14z"/>
      <path d="M14 16.5L11 14h23l-3 2.5H14zM11 14V9h4v2h5V9h5v2h5V9h4v5H11z"/>
      <path d="M12 35.5h21M13 31.5h19M14 29.5h17M14 16.5h17M11 14h23" stroke="${d}" fill="none"/>
    </g>`, true),
  B: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <g fill="none" stroke-linejoin="miter">
        <path d="M9 36c3.39-.97 10.11.43 13.5-2 3.39 2.43 10.11 1.03 13.5 2 0 0 1.65.54 3 2-.68.97-1.65.99-3 .5-3.39-.97-10.11.46-13.5-1-3.39 1.46-10.11.03-13.5 1-1.354.49-2.323.47-3-.5 1.354-1.94 3-2 3-2z"/>
        <path d="M15 32c2.5 2.5 12.5 2.5 15 0 .5-1.5 0-2 0-2 0-2.5-2.5-4-2.5-4 5.5-1.5 6-11.5-5-15.5-11 4-10.5 14-5 15.5 0 0-2.5 1.5-2.5 4 0 0-.5.5 0 2z"/>
        <path d="M25 8a2.5 2.5 0 1 1-5 0 2.5 2.5 0 1 1 5 0z"/>
      </g>
      <path d="M17.5 26h10M15 30h15" stroke="${d}" fill="none" stroke-linejoin="miter"/>
    </g>`, true),
  N: pieceSVG.bind(null, (f,s,d) => `
    <g fill="none" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M22 10c10.5 1 16.5 8 16 29H15c0-9 10-6.5 8-21" fill="${f}"/>
      <path d="M24 18c.38 2.91-5.55 7.37-8 9-3 2-2.82 4.34-5 4-1.042-.94 1.41-3.04 0-3-1 0 .19 1.23-1 2-1 0-4.003 1-4-4 0-2 6-12 6-12s1.89-1.9 2-3.5c-.73-.994-.5-2-.5-3 1-1 3 2.5 3 2.5h2s.78-1.992 2.5-3c1 0 1 3 1 3" fill="${f}"/>
      <path d="M9.5 25.5a.5.5 0 1 1-1 0 .5.5 0 1 1 1 0z" fill="${s}"/>
      <path d="M14.933 15.75a.5 1.5 30 1 1-.866-.5.5 1.5 30 1 1 .866.5z" fill="${s}"/>
    </g>`, true),
  P: pieceSVG.bind(null, (f,s,d) => `
    <path d="M22.5 9c-2.21 0-4 1.79-4 4 0 .89.29 1.71.78 2.38C17.33 16.5 16 18.59 16 21c0 2.03.94 3.84 2.41 5.03C15.41 27.09 11 31.58 11 39.5H34c0-7.92-4.41-12.41-7.41-13.47C28.06 24.84 29 23.03 29 21c0-2.41-1.33-4.5-3.28-5.62.49-.67.78-1.49.78-2.38 0-2.21-1.79-4-4-4z" fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round"/>`, true),
  k: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M22.5 11.63V6" stroke="${s}"/><path d="M20 8h5" stroke="${s}"/>
      <path d="M22.5 25s4.5-7.5 3-10.5c0 0-1-2.5-3-2.5s-3 2.5-3 2.5c-1.5 3 3 10.5 3 10.5"/>
      <path d="M12.5 37c5.5 3.5 14.5 3.5 20 0v-7s9-4.5 6-10.5c-4-6.5-13.5-3.5-16 4V27v-3.5c-3.5-7.5-13-10.5-16-4-3 6 5 10 5 10V37z"/>
      <path d="M12.5 30c5.5-3 14.5-3 20 0M12.5 33.5c5.5-3 14.5-3 20 0M12.5 37c5.5-3 14.5-3 20 0" stroke="${d}"/>
    </g>`, false),
  q: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M9 26c8.5-1.5 21-1.5 27 0l2.5-12.5L31 25l-3.5-12.5-5.5 8-5.5-8L13 25 5.5 13.5 9 26z" stroke-linecap="butt"/>
      <path d="M9 26c0 2 1.5 2 2.5 4 1 1.5 1 1 .5 3.5-1.5 1-1.5 2.5-1.5 2.5-1.5 1.5.5 2.5.5 2.5 6.5 1 16.5 1 23 0 0 0 1.5-1 0-2.5 0 0 .5-1.5-1-2.5-.5-2.5-.5-2 .5-3.5 1-2 2.5-2 2.5-4-8.5-1.5-18.5-1.5-27 0z" stroke-linecap="butt"/>
      <path d="M11.5 30c3.5-1 18.5-1 22 0M12 33.5c6-1 15-1 21 0" stroke="${d}" fill="none"/>
      <circle cx="6" cy="12" r="2"/><circle cx="14" cy="9" r="2"/><circle cx="22.5" cy="8" r="2"/><circle cx="31" cy="9" r="2"/><circle cx="39" cy="12" r="2"/>
    </g>`, false),
  r: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M9 39h27v-3H9v3zM12.5 32l1.5-2.5h17l1.5 2.5h-20zM12 36v-4h21v4H12z"/>
      <path d="M14 29.5v-13h17v13H14z"/>
      <path d="M14 16.5L11 14h23l-3 2.5H14zM11 14V9h4v2h5V9h5v2h5V9h4v5H11z"/>
      <path d="M12 35.5h21M13 31.5h19M14 29.5h17M14 16.5h17M11 14h23" stroke="${d}" fill="none"/>
    </g>`, false),
  b: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <g fill="none" stroke-linejoin="miter">
        <path d="M9 36c3.39-.97 10.11.43 13.5-2 3.39 2.43 10.11 1.03 13.5 2 0 0 1.65.54 3 2-.68.97-1.65.99-3 .5-3.39-.97-10.11.46-13.5-1-3.39 1.46-10.11.03-13.5 1-1.354.49-2.323.47-3-.5 1.354-1.94 3-2 3-2z"/>
        <path d="M15 32c2.5 2.5 12.5 2.5 15 0 .5-1.5 0-2 0-2 0-2.5-2.5-4-2.5-4 5.5-1.5 6-11.5-5-15.5-11 4-10.5 14-5 15.5 0 0-2.5 1.5-2.5 4 0 0-.5.5 0 2z"/>
        <path d="M25 8a2.5 2.5 0 1 1-5 0 2.5 2.5 0 1 1 5 0z"/>
      </g>
      <path d="M17.5 26h10M15 30h15" stroke="${d}" fill="none" stroke-linejoin="miter"/>
    </g>`, false),
  n: pieceSVG.bind(null, (f,s,d) => `
    <g fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M22 10c10.5 1 16.5 8 16 29H15c0-9 10-6.5 8-21" fill="${f}"/>
      <path d="M24 18c.38 2.91-5.55 7.37-8 9-3 2-2.82 4.34-5 4-1.042-.94 1.41-3.04 0-3-1 0 .19 1.23-1 2-1 0-4.003 1-4-4 0-2 6-12 6-12s1.89-1.9 2-3.5c-.73-.994-.5-2-.5-3 1-1 3 2.5 3 2.5h2s.78-1.992 2.5-3c1 0 1 3 1 3" fill="${f}"/>
      <path d="M9.5 25.5a.5.5 0 1 1-1 0 .5.5 0 1 1 1 0z" fill="#fff"/>
      <path d="M14.933 15.75a.5 1.5 30 1 1-.866-.5.5 1.5 30 1 1 .866.5z" fill="#fff"/>
    </g>`, false),
  p: pieceSVG.bind(null, (f,s,d) => `
    <path d="M22.5 9c-2.21 0-4 1.79-4 4 0 .89.29 1.71.78 2.38C17.33 16.5 16 18.59 16 21c0 2.03.94 3.84 2.41 5.03C15.41 27.09 11 31.58 11 39.5H34c0-7.92-4.41-12.41-7.41-13.47C28.06 24.84 29 23.03 29 21c0-2.41-1.33-4.5-3.28-5.62.49-.67.78-1.49.78-2.38 0-2.21-1.79-4-4-4z" fill="${f}" stroke="${s}" stroke-width="1.5" stroke-linecap="round"/>`, false),
};
```

Note: Each entry calls `pieceSVG.bind(null, pathFn, isWhite)`. When called with no arguments, `pieceSVG(pathFn, isWhite)` generates the full SVG string.

- [ ] **Step 2: Update renderBoard() to use SVG**

Replace the piece rendering logic inside `renderBoard()` (line 912-913). Instead of setting `textContent` with Unicode, set the piece via DOM.

In `index.html`, in the `renderBoard` function, replace:
```js
      const piece = board[r][c];
      if (piece) sq.textContent = PIECE_SYMBOLS[piece];
```

With:
```js
      const piece = board[r][c];
      if (piece && PIECE_SVG[piece]) {
        const svgWrapper = document.createElement('div');
        svgWrapper.style.cssText = 'width:80%;height:80%;display:flex;align-items:center;justify-content:center;pointer-events:none;';
        svgWrapper.innerHTML = PIECE_SVG[piece]();
        sq.appendChild(svgWrapper);
      }
```

Note: The SVG content is from the hardcoded PIECE_SVG object (no user input), so this innerHTML usage is safe.

- [ ] **Step 3: Add CSS for SVG pieces in squares**

In the `<style>` block, after the `.square` rules (around line 36), add:

```css
.square svg { pointer-events: none; }
```

- [ ] **Step 4: Verify SVG pieces render correctly**

Open `http://127.0.0.1:8766/index.html` in the browser. Check:
- All 12 piece types visible on the initial board
- White pieces clearly visible on light squares (white fill + dark stroke)
- Black pieces clearly visible on dark squares (dark fill + dark stroke)
- Pieces scale correctly on mobile viewport (resize browser to 400px width)
- Clicking squares in practice mode still works (pointer-events:none on SVG)

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Replace Unicode chess pieces with inline SVG for better visibility"
```

---

### Task 2: Add `playerSide` field and auto-play opponent logic

**Files:**
- Modify: `index.html:294-863` (add `playerSide` to each opening in OPENINGS)
- Modify: `index.html:1023-1041` (update `startPractice()`)
- Modify: `index.html:1084-1141` (update `onPracticeClick()`)

The practice mode currently requires the user to play both sides. This task makes the opponent auto-play so the user only operates their side.

- [ ] **Step 1: Add `playerSide` to all OPENINGS entries**

Add `playerSide: "white"` or `playerSide: "black"` to each opening object. The rule: openings with "防御" or "Defense" in the name get `"black"` (user plays the defender), all others get `"white"`.

Openings that get `playerSide: "black"`:
- 法兰西防御 (French Defense)
- 卡罗-卡恩防御 (Caro-Kann Defense)
- 西西里防御 (Sicilian Defense)
- 西西里防御-龙式变化 (Sicilian Dragon)
- 西西里防御-纳伊道夫变化 (Sicilian Najdorf)
- 斯堪的纳维亚防御 (Scandinavian Defense)
- 皮尔茨防御 (Pirc Defense)
- 尼姆佐-印度防御 (Nimzo-Indian Defense)
- 国王印度防御 (King's Indian Defense)
- 格林菲尔德防御 (Grünfeld Defense)
- 荷兰防御 (Dutch Defense)
- 阿廖欣防御 (Alekhine's Defense)
- 斯拉夫防御 (Slav Defense)

All others get `playerSide: "white"`:
- 意大利开局, 西班牙开局, 苏格兰开局, 后翼弃兵, 后翼弃兵接受, 英国开局, 列蒂开局, 国王印度进攻, 飞象开局, 伦敦体系, 王翼弃兵, 维也纳开局

Add the field right after the `category` field in each opening. For example:
```js
{
  name: "意大利开局", nameEn: "Italian Game", eco: "C50-C54",
  category: "e4 开局",
  playerSide: "white",
  moves: [...],
  ...
}
```

- [ ] **Step 2: Add autoPlayOpponent() function**

Add a new function after the `onPracticeClick` function (after line 1141):

```js
let autoPlayTimeout = null;

function autoPlayOpponent() {
  if (!practiceState.opening) return;
  if (practiceState.moveIndex >= practiceState.opening.moves.length) return;

  const opening = practiceState.opening;
  const playerSide = opening.playerSide || 'white';
  const isWhiteTurn = practiceState.moveIndex % 2 === 0;
  const isPlayerTurn = (playerSide === 'white' && isWhiteTurn) || (playerSide === 'black' && !isWhiteTurn);

  if (isPlayerTurn) return; // Player's turn, don't auto-play

  // Disable board clicks during auto-play
  practiceState.autoPlaying = true;

  autoPlayTimeout = setTimeout(() => {
    const move = opening.moves[practiceState.moveIndex];
    const justCompletedIdx = practiceState.moveIndex;

    practiceState.board = applyMove(practiceState.board, move);
    practiceState.moveIndex++;
    practiceState.maxMoveIndex = practiceState.moveIndex;
    renderBoard('practice-board', practiceState.board, onPracticeClick);

    // Highlight the auto-played move destination
    const tc = move.charCodeAt(2) - 97, tr = 8 - parseInt(move[3]);
    setHighlight('practice-board', tr, tc, true);

    updatePracticeProgress();
    renderPracticeMoves();
    autoExplainMove(justCompletedIdx);
    updateNavButtons();

    practiceState.autoPlaying = false;

    // Check if practice is complete
    if (practiceState.moveIndex >= opening.moves.length) {
      showFeedback('practice-feedback', `完美！你完成了「${opening.name}」的所有走法！`, 'success');
      return;
    }

    // If still opponent's turn (shouldn't happen in normal openings, but safety)
    autoPlayOpponent();
  }, 500);
}
```

- [ ] **Step 3: Update startPractice() to trigger auto-play**

In `startPractice()` (line 1025), replace the full function with:

```js
function startPractice(opening) {
  if (autoPlayTimeout) { clearTimeout(autoPlayTimeout); autoPlayTimeout = null; }
  practiceState = { opening, board: cloneBoard(INITIAL_BOARD), moveIndex: 0, selected: null, streak: practiceState.streak, autoPlaying: false, maxMoveIndex: 0, browsing: false };
  document.getElementById('practice-name').textContent = opening.name.length > 12 ? opening.name.substring(0,12)+'...' : opening.name;
  document.getElementById('practice-title').textContent = opening.name;
  document.getElementById('practice-desc').textContent = opening.desc;
  updatePracticeProgress();
  renderBoard('practice-board', practiceState.board, onPracticeClick);
  renderPracticeMoves();
  hideFeedback('practice-feedback');
  hideAllTips();
  // Hide demo panel, reset for new opening
  document.getElementById('demo-panel').style.display = 'none';
  document.getElementById('demo-messages').innerHTML = '';
  document.getElementById('demo-nav').style.display = 'none';
  demoPlaying = false;
  demoState = null;
  updateNavButtons();

  // Auto-play first move if it's opponent's turn
  autoPlayOpponent();
}
```

Note: demo-messages innerHTML is clearing internal UI state, not rendering user input.

- [ ] **Step 4: Update onPracticeClick() to block during auto-play and trigger next auto-play**

In `onPracticeClick()`, add a guard at the top and trigger auto-play after a correct move.

Add after line 1086 (`if (demoPlaying) return;`):
```js
  if (practiceState.autoPlaying) return;
```

In the correct-move block, after `practiceState.moveIndex++;` add:
```js
      practiceState.maxMoveIndex = practiceState.moveIndex;
      practiceState.browsing = false;
```

After `hideFeedback('practice-feedback');` (in the else branch for not-complete), add:
```js
      // Trigger opponent auto-play if needed
      autoPlayOpponent();
```

Also add `updateNavButtons();` after `updatePracticeProgress();` in the correct block.

- [ ] **Step 5: Update the turn indicator message**

In `onPracticeClick()`, update the wrong-side message at line 1098. Replace:
```js
      showFeedback('practice-feedback', isWhiteTurn ? '现在轮到白方走棋，请选择白色棋子' : '现在轮到黑方走棋，请选择黑色棋子', 'info');
```

With:
```js
      const playerSide = practiceState.opening.playerSide || 'white';
      const playerColor = playerSide === 'white' ? '白色' : '黑色';
      showFeedback('practice-feedback', `请选择${playerColor}棋子——你扮演${playerColor === '白色' ? '白方' : '黑方'}`, 'info');
```

- [ ] **Step 6: Verify auto-play works**

Open `http://127.0.0.1:8766/index.html`, go to Practice mode:
1. Select "意大利开局" (playerSide: white) — user should play white, black auto-plays after each white move
2. Select "西西里防御" (playerSide: black) — white should auto-play first (e4), then user plays black (c5)
3. Verify opponent moves show after 500ms delay with highlight
4. Verify board is not clickable during auto-play animation
5. Verify speech bubbles show explanations for auto-played moves

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Add auto-play opponent in practice mode, user only plays opening side"
```

---

### Task 3: Add move navigation controls

**Files:**
- Modify: `index.html:200-202` (add navigation buttons in HTML)
- Modify: `index.html:1023` (update practiceState — already done in Task 2)
- Add new functions: `navigateBack()`, `navigateForward()`, `updateNavButtons()`, keyboard handler

Move history tracking + navigation for reviewing moves in practice mode.

- [ ] **Step 1: Add navigation buttons to HTML**

In `index.html`, replace the progress bar section (line 202):
```html
        <div class="progress-bar"><div class="progress-fill" id="practice-bar"></div></div>
```

With:
```html
        <div style="display:flex;align-items:center;gap:8px;margin-top:8px;">
          <button class="btn btn-secondary" id="nav-back" onclick="navigateBack()" style="padding:6px 14px;font-size:1.1em;" title="后退" disabled>←</button>
          <div class="progress-bar" style="flex:1;"><div class="progress-fill" id="practice-bar"></div></div>
          <button class="btn btn-secondary" id="nav-forward" onclick="navigateForward()" style="padding:6px 14px;font-size:1.1em;" title="前进" disabled>→</button>
        </div>
```

- [ ] **Step 2: Add navigateBack(), navigateForward(), and updateNavButtons() functions**

Add these after the `autoPlayOpponent()` function:

```js
function navigateBack() {
  if (!practiceState.opening) return;
  if (demoPlaying) return;
  if (practiceState.moveIndex <= 0) return;
  if (practiceState.autoPlaying) return;

  // Cancel any pending auto-play
  if (autoPlayTimeout) { clearTimeout(autoPlayTimeout); autoPlayTimeout = null; }

  practiceState.browsing = true;
  practiceState.selected = null;
  practiceState.moveIndex--;

  // Rebuild board from scratch up to current index
  practiceState.board = boardAfterMoves(practiceState.opening.moves.slice(0, practiceState.moveIndex));
  renderBoard('practice-board', practiceState.board, onPracticeClick);
  clearHighlights('practice-board');

  // Highlight the last move destination if there is one
  if (practiceState.moveIndex > 0) {
    const lastMove = practiceState.opening.moves[practiceState.moveIndex - 1];
    const tc = lastMove.charCodeAt(2) - 97, tr = 8 - parseInt(lastMove[3]);
    setHighlight('practice-board', tr, tc, true);
  }

  updatePracticeProgress();
  renderPracticeMoves();
  hideAllTips();
  hideFeedback('practice-feedback');
  updateNavButtons();
}

function navigateForward() {
  if (!practiceState.opening) return;
  if (demoPlaying) return;
  if (practiceState.autoPlaying) return;
  if (practiceState.moveIndex >= practiceState.maxMoveIndex) return;

  practiceState.moveIndex++;

  // Rebuild board
  practiceState.board = boardAfterMoves(practiceState.opening.moves.slice(0, practiceState.moveIndex));
  renderBoard('practice-board', practiceState.board, onPracticeClick);
  clearHighlights('practice-board');

  // Highlight last move
  const lastMove = practiceState.opening.moves[practiceState.moveIndex - 1];
  const tc = lastMove.charCodeAt(2) - 97, tr = 8 - parseInt(lastMove[3]);
  setHighlight('practice-board', tr, tc, true);

  // Show explanation
  autoExplainMove(practiceState.moveIndex - 1);

  // Check if we're caught up
  if (practiceState.moveIndex >= practiceState.maxMoveIndex) {
    practiceState.browsing = false;
    // Trigger auto-play if needed
    autoPlayOpponent();
  }

  updatePracticeProgress();
  renderPracticeMoves();
  updateNavButtons();
}

function updateNavButtons() {
  const backBtn = document.getElementById('nav-back');
  const fwdBtn = document.getElementById('nav-forward');
  if (backBtn) backBtn.disabled = !practiceState.opening || practiceState.moveIndex <= 0;
  if (fwdBtn) fwdBtn.disabled = !practiceState.opening || practiceState.moveIndex >= practiceState.maxMoveIndex;
}
```

- [ ] **Step 3: Add keyboard handler**

Add after the navigation functions:

```js
document.addEventListener('keydown', (e) => {
  // Only handle when practice panel is active
  if (!document.getElementById('panel-practice').classList.contains('active')) return;
  if (e.key === 'ArrowLeft') { e.preventDefault(); navigateBack(); }
  if (e.key === 'ArrowRight') { e.preventDefault(); navigateForward(); }
});
```

- [ ] **Step 4: Handle browsing state in onPracticeClick()**

At the top of `onPracticeClick()`, after the `autoPlaying` guard, add:
```js
  // If browsing history, clicking a piece returns to current position first
  if (practiceState.browsing) {
    practiceState.moveIndex = practiceState.maxMoveIndex;
    practiceState.board = boardAfterMoves(practiceState.opening.moves.slice(0, practiceState.moveIndex));
    renderBoard('practice-board', practiceState.board, onPracticeClick);
    practiceState.browsing = false;
    updateNavButtons();
    updatePracticeProgress();
    renderPracticeMoves();
    autoPlayOpponent();
    return;
  }
```

- [ ] **Step 5: Verify navigation works**

1. Start a practice session (e.g., "意大利开局")
2. Play 3 correct moves
3. Press left arrow key — board should go back one move
4. Press left arrow again — back another step
5. Press right arrow — forward one step (auto-plays if opponent)
6. Click a piece while browsing — should return to current position
7. Verify ← button disabled at start, → button disabled at max position

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Add move navigation with back/forward buttons and arrow key shortcuts"
```

---

### Task 4: Add mistakes data to all openings

**Files:**
- Modify: `index.html:294-863` (add `mistakes` array to each OPENINGS entry)

This is a data-only task. Add 1-2 common mistakes per opening with explanations and punishment sequences. This data powers the error feedback feature in Task 5.

- [ ] **Step 1: Add mistakes to e4 openings (Italian through Alekhine)**

Add `mistakes` array to each opening, after the `quiz` field. Each mistake object has: `atMove` (0-indexed move where the mistake occurs), `wrong` (the wrong move string in the same format as the moves array), `explain` (why it's wrong), `punishment` (opponent exploitation moves), `punchline` (summary of the consequence).

```js
// Italian Game (意大利开局):
mistakes: [
  {
    atMove: 4,
    wrong: "f1b5",
    explain: "Bb5 是西班牙开局的思路而非意大利。Bc4 瞄准 f7 弱点更直接，f7 只有黑王防守。",
    punishment: ["a7a6","b5a4","d7d5"],
    punchline: "黑方用 a6 驱赶白象，再 d5 反击中心，白方失去进攻节奏"
  },
  {
    atMove: 2,
    wrong: "f1c4",
    explain: "过早出象！应该先 Nf3 攻击 e5 兵。Bc4 虽然也是发展，但没有对黑方形成任何威胁。",
    punishment: ["g8f6","d2d3","f8c5"],
    punchline: "黑方从容发展，白方失去了先手攻击 e5 的机会"
  }
]

// Ruy Lopez (西班牙开局):
mistakes: [
  {
    atMove: 4,
    wrong: "f1c4",
    explain: "Bc4 是意大利开局，不是西班牙。Bb5 对 c6 马施压，间接威胁 e5 兵，战略更深远。",
    punishment: ["f8c5","d2d3","d7d6"],
    punchline: "走入意大利对称局面，白方失去了西班牙开局的长期施压优势"
  }
]

// Scotch Game (苏格兰开局):
mistakes: [
  {
    atMove: 4,
    wrong: "f1c4",
    explain: "Bc4 是意大利风格。苏格兰的核心是 d4 立刻冲击中心。",
    punishment: ["f8c5","d2d3","d7d6"],
    punchline: "进入慢速局面，丧失苏格兰的主动权"
  },
  {
    atMove: 4,
    wrong: "f1b5",
    explain: "Bb5 是西班牙开局。苏格兰要的是 d4 快速打开局面。",
    punishment: ["a7a6","b5a4","d7d5"],
    punchline: "黑方轻松反击中心，白方两头不着"
  }
]

// French Defense (法兰西防御):
mistakes: [
  {
    atMove: 1,
    wrong: "d7d5",
    explain: "先 d5 是斯堪的纳维亚防御！法兰西先 e6 为 d5 提供支撑，中心更稳固。",
    punishment: ["e4d5","d8d5","b1c3"],
    punchline: "黑后被迫出来吃兵，然后被 Nc3 驱赶，浪费时间"
  }
]

// Caro-Kann Defense (卡罗-卡恩防御):
mistakes: [
  {
    atMove: 1,
    wrong: "e7e6",
    explain: "e6 是法兰西防御！卡罗-卡恩走 c6 不封住 c8 象出路。走 e6 后象被困在兵链内。",
    punishment: ["d2d4","d7d5","b1c3"],
    punchline: "进入法兰西结构，c8 象被封死，失去卡罗-卡恩的核心优势"
  }
]

// Sicilian Defense (西西里防御):
mistakes: [
  {
    atMove: 1,
    wrong: "e7e5",
    explain: "e5 是对称应对，局面平稳。西西里走 c5 制造不对称局面，给黑方更多赢棋机会。",
    punishment: ["g1f3","b8c6","f1b5"],
    punchline: "进入对称开局，黑方作为后手很难争取主动"
  }
]

// Sicilian Dragon (西西里龙式):
mistakes: [
  {
    atMove: 9,
    wrong: "f8g7",
    explain: "顺序错！应先 g6 再 Bg7。g6 是龙式的定义手，直接 Bg7 不合法（g7有兵）。",
    punishment: ["f2f4","f8g7","e4e5"],
    punchline: "白方 f4+e5 推进，龙式大炮哑火"
  }
]

// Sicilian Najdorf (西西里纳伊道夫):
mistakes: [
  {
    atMove: 9,
    wrong: "e7e5",
    explain: "直接 e5 过早暴露意图。纳伊道夫先 a6 保持灵活性，保留 e5/e6/g6 各种选择。",
    punishment: ["d4b3","f8e7","f2f4"],
    punchline: "白方获得 f4 反击机会，黑方锁定兵结构失去灵活性"
  }
]

// Scandinavian Defense (斯堪的纳维亚防御):
mistakes: [
  {
    atMove: 1,
    wrong: "e7e5",
    explain: "e5 是对称应对。斯堪的纳维亚的特点是 d5 直接挑战 e4。",
    punishment: ["g1f3","b8c6","f1b5"],
    punchline: "进入普通对称局面，没有利用黑方积极反击的机会"
  }
]

// Pirc Defense (皮尔茨防御):
mistakes: [
  {
    atMove: 1,
    wrong: "g8f6",
    explain: "先 Nf6 是阿廖欣防御的节奏。皮尔茨应先 d6 稳固。",
    punishment: ["e4e5","f6d5","d2d4"],
    punchline: "白方 e5 驱赶马建立强大中心，黑方节奏被打乱"
  }
]

// Alekhine's Defense (阿廖欣防御):
mistakes: [
  {
    atMove: 1,
    wrong: "e7e5",
    explain: "e5 是标准对称应对。阿廖欣走 Nf6 挑衅白方推兵，这才是超现代精髓。",
    punishment: ["g1f3","b8c6","f1b5"],
    punchline: "进入对称开局，失去阿廖欣的挑衅性反击机会"
  }
]
```

- [ ] **Step 2: Add mistakes to d4 openings (Queen's Gambit through Dutch)**

```js
// Queen's Gambit (后翼弃兵):
mistakes: [
  {
    atMove: 2,
    wrong: "g1f3",
    explain: "Nf3 不是后翼弃兵。c4 才是核心——用侧翼兵挑战中心。",
    punishment: ["g8f6","c1f4","e7e6"],
    punchline: "进入伦敦或其他 d4 开局，失去后翼弃兵的中心施压"
  }
]

// Queen's Gambit Accepted (后翼弃兵接受):
mistakes: [
  {
    atMove: 3,
    wrong: "c7c6",
    explain: "c6 是斯拉夫防御！接受弃兵要 dxc4 吃掉白方的兵。",
    punishment: ["g1f3","g8f6","b1c3"],
    punchline: "进入斯拉夫防御，两个开局策略完全不同"
  }
]

// Slav Defense (斯拉夫防御):
mistakes: [
  {
    atMove: 3,
    wrong: "e7e6",
    explain: "e6 是正统防御而非斯拉夫！斯拉夫走 c6 不封住 c8 象。走 e6 后象被困。",
    punishment: ["b1c3","g8f6","c1g5"],
    punchline: "进入后翼弃兵正统防御，c8 象被封死"
  }
]

// Nimzo-Indian Defense (尼姆佐-印度防御):
mistakes: [
  {
    atMove: 5,
    wrong: "f8e7",
    explain: "Be7 太消极！Bb4 钉住 c3 马才是尼姆佐精髓——阻止白方 e4 扩大中心。",
    punishment: ["e2e4","d7d5","e4e5"],
    punchline: "白方顺利推 e4 建立大中心，黑方失去控制 e4 的机会"
  }
]

// King's Indian Defense (国王印度防御):
mistakes: [
  {
    atMove: 3,
    wrong: "d7d5",
    explain: "d5 是格林菲尔德防御！国王印度应先 g6+Bg7 建立超现代布局。",
    punishment: ["c4d5","f6d5","e2e4"],
    punchline: "进入格林菲尔德而非国王印度，战略方向完全不同"
  },
  {
    atMove: 7,
    wrong: "c7c5",
    explain: "过早反击！应先 d6 稳固。c5 在没准备时让白方 d5 推进占据空间。",
    punishment: ["d4d5","f6a6","e2e4"],
    punchline: "白方 d5 封住中心获得巨大空间优势"
  }
]

// Grunfeld Defense (格林菲尔德防御):
mistakes: [
  {
    atMove: 5,
    wrong: "f8g7",
    explain: "先 Bg7 是国王印度的节奏！格林菲尔德此刻走 d5 直接轰击白方中心。",
    punishment: ["e2e4","d7d6","f1e2"],
    punchline: "白方轻松建立大中心，黑方进入被动的国王印度结构"
  }
]

// Dutch Defense (荷兰防御):
mistakes: [
  {
    atMove: 1,
    wrong: "d7d5",
    explain: "d5 是后翼弃兵框架！荷兰的核心是 f5 控制 e4 并准备王翼进攻。",
    punishment: ["c2c4","e7e6","b1c3"],
    punchline: "进入后翼弃兵结构，失去荷兰防御的进攻性"
  }
]
```

- [ ] **Step 3: Add mistakes to flank openings and remaining**

```js
// English Opening (英国开局):
mistakes: [
  {
    atMove: 0,
    wrong: "e2e4",
    explain: "e4 是王兵开局！英国开局的特点是 c4 从侧翼控制 d5。",
    punishment: ["e7e5","g1f3","b8c6"],
    punchline: "进入普通 e4 e5 开局，失去侧翼控制策略"
  }
]

// Reti Opening (列蒂开局):
mistakes: [
  {
    atMove: 2,
    wrong: "g2g3",
    explain: "g3 是国王印度进攻的思路。列蒂核心是 c4 从侧翼挑战已占据的 d5 中心。",
    punishment: ["c7c5","f1g2","b8c6"],
    punchline: "黑方稳固中心，白方失去挑战 d5 的最佳时机"
  }
]

// King's Indian Attack (国王印度进攻):
mistakes: [
  {
    atMove: 2,
    wrong: "c2c4",
    explain: "c4 是列蒂开局！国王印度进攻走 g3 准备 Bg2 侧翼发展体系。",
    punishment: ["d5c4","b1a3","c7c5"],
    punchline: "黑方接受弃兵，白方进入不熟悉的列蒂变化"
  }
]

// Catalan Opening (飞象开局):
mistakes: [
  {
    atMove: 4,
    wrong: "b1c3",
    explain: "Nc3 是常规发展但不是飞象！g3 才是飞象标志——结合 d4 体系和侧翼象。",
    punishment: ["f8b4","d1c2","d7d5"],
    punchline: "黑方走入尼姆佐钉住马，白方失去飞象的长对角线压力"
  }
]

// London System (伦敦体系):
mistakes: [
  {
    atMove: 2,
    wrong: "c2c4",
    explain: "c4 是后翼弃兵！伦敦标志是 Bf4——先出象再走 e3，否则 e3 封住象出路。",
    punishment: ["d5c4","g1f3","g8f6"],
    punchline: "黑方接受弃兵，进入后翼弃兵而非伦敦体系"
  }
]

// King's Gambit (王翼弃兵):
mistakes: [
  {
    atMove: 2,
    wrong: "g1f3",
    explain: "Nf3 是最常见的但不是王翼弃兵！f4 才是——主动弃兵换中心控制和 f 线进攻。",
    punishment: ["b8c6","f1c4","f8c5"],
    punchline: "进入安静的意大利类局面，失去王翼弃兵的浪漫攻势"
  }
]

// Vienna Game (维也纳开局):
mistakes: [
  {
    atMove: 2,
    wrong: "g1f3",
    explain: "Nf3 是最普通的发展。Nc3 才是维也纳——保留 f4 推进的选择权。",
    punishment: ["b8c6","f1c4","f8c5"],
    punchline: "进入意大利标准局面，白方失去 f4 弃兵的选择权"
  }
]
```

- [ ] **Step 4: Verify all openings have mistakes**

Count the openings in OPENINGS array and verify each has a `mistakes` array with at least one entry. There should be 25 openings total.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add mistakes data for all 25 openings with explanations and punishment sequences"
```

---

### Task 5: Implement error feedback with consequence demo

**Files:**
- Modify: `index.html` CSS block (add `.speech-bubble.error-bubble` styles)
- Modify: `index.html` `onPracticeClick()` (replace wrong-move handler)
- Add new functions: `showMistakeFeedback()`, `playPunishment()`, `revertFromPunishment()`

This is the core teaching feature. When the user makes a wrong move, the speech bubble turns red with an explanation and a "see consequences" button.

- [ ] **Step 1: Add error bubble CSS**

In the `<style>` block, after the `.speech-bubble.black-bubble::after` rule (around line 125), add:

```css
.speech-bubble.error-bubble { border-color: #f44336; background: rgba(60,20,20,0.95); }
.speech-bubble.error-bubble::after { border-left-color: #f44336; }
.speech-bubble.error-bubble .tip-side { color: #f44336; }
.mistake-btn { display: inline-block; margin-top: 8px; padding: 6px 14px; background: #e0c068; color: #1a1a2e; border: none; border-radius: 6px; cursor: pointer; font-size: 0.82em; font-weight: 600; width: 100%; text-align: center; transition: background 0.2s; }
.mistake-btn:hover { background: #c8a84e; }
```

- [ ] **Step 2: Add showMistakeFeedback() function**

Add after the `autoExplainMove` function:

```js
let punishmentState = null;

function showMistakeFeedback(wrongMoveStr) {
  const opening = practiceState.opening;
  if (!opening) return;

  const moveIdx = practiceState.moveIndex;
  const isWhite = moveIdx % 2 === 0;
  const side = isWhite ? 'white' : 'black';
  const sideLabel = isWhite ? '❌ 白方' : '❌ 黑方';

  // Find matching mistake in data
  const mistake = opening.mistakes && opening.mistakes.find(
    m => m.atMove === moveIdx && m.wrong === wrongMoveStr
  );

  const tip = document.getElementById(side === 'white' ? 'tip-white' : 'tip-black');

  if (mistake) {
    // Show detailed feedback with punishment button
    const btnHTML = '<button class="mistake-btn" onclick="playPunishment()">&#x1F441; 看看后果</button>';
    const dismissHTML = '<span class="tip-dismiss">点击泡泡关闭</span>';
    tip.innerHTML = '';
    const sideSpan = document.createElement('span');
    sideSpan.className = 'tip-side';
    sideSpan.textContent = sideLabel;
    tip.appendChild(sideSpan);
    tip.appendChild(document.createTextNode(' ' + mistake.explain));

    const btn = document.createElement('button');
    btn.className = 'mistake-btn';
    btn.textContent = '\u{1F441} 看看后果';
    btn.onclick = playPunishment;
    tip.appendChild(btn);

    const dismiss = document.createElement('span');
    dismiss.className = 'tip-dismiss';
    dismiss.textContent = '点击泡泡关闭';
    tip.appendChild(dismiss);

    tip.className = 'speech-bubble error-bubble show';

    // Store punishment data for playback
    punishmentState = {
      mistake: mistake,
      boardBeforeWrong: cloneBoard(practiceState.board),
      wrongMove: wrongMoveStr
    };
  } else {
    // Generic feedback — show correct move explanation
    const correctMove = opening.moves[moveIdx];
    const explain = opening.moveExplain && opening.moveExplain[moveIdx]
      ? opening.moveExplain[moveIdx] : '';

    tip.innerHTML = '';
    const sideSpan = document.createElement('span');
    sideSpan.className = 'tip-side';
    sideSpan.textContent = sideLabel;
    tip.appendChild(sideSpan);
    tip.appendChild(document.createTextNode(
      ' 这步不是主线走法。正确走法：' + moveToNotation(correctMove, moveIdx)
    ));
    if (explain) {
      const small = document.createElement('small');
      small.style.color = '#aaa';
      small.style.display = 'block';
      small.style.marginTop = '4px';
      small.textContent = explain;
      tip.appendChild(small);
    }
    const dismiss = document.createElement('span');
    dismiss.className = 'tip-dismiss';
    dismiss.textContent = '点击泡泡关闭';
    tip.appendChild(dismiss);

    tip.className = 'speech-bubble error-bubble show';
    punishmentState = null;
  }
}
```

- [ ] **Step 3: Add playPunishment() and revertFromPunishment() functions**

Add after `showMistakeFeedback()`:

```js
function playPunishment() {
  if (!punishmentState) return;
  const { mistake, boardBeforeWrong, wrongMove } = punishmentState;

  // First apply the wrong move
  let board = applyMove(boardBeforeWrong, wrongMove);
  const boards = [board];
  // Then apply punishment moves
  for (const m of mistake.punishment) {
    board = applyMove(board, m);
    boards.push(cloneBoard(board));
  }

  // Disable interaction
  practiceState.autoPlaying = true;
  hideAllTips();

  // Show the wrong move first
  renderBoard('practice-board', boards[0], onPracticeClick);
  const wtc = wrongMove.charCodeAt(2) - 97, wtr = 8 - parseInt(wrongMove[3]);
  highlightSquare('practice-board', wtr, wtc, 'wrong');

  // Animate punishment moves one by one
  let step = 0;
  const interval = setInterval(() => {
    if (step >= mistake.punishment.length) {
      clearInterval(interval);
      // Show punchline and revert button
      const isWhite = practiceState.moveIndex % 2 === 0;
      const side = isWhite ? 'white' : 'black';
      const tip = document.getElementById(side === 'white' ? 'tip-white' : 'tip-black');

      tip.innerHTML = '';
      const sideSpan = document.createElement('span');
      sideSpan.className = 'tip-side';
      sideSpan.style.color = '#f44336';
      sideSpan.textContent = '\u{1F4A1} 后果';
      tip.appendChild(sideSpan);
      tip.appendChild(document.createTextNode(' ' + mistake.punchline));

      const btn = document.createElement('button');
      btn.className = 'mistake-btn';
      btn.textContent = '\u21A9 回到正确位置';
      btn.onclick = revertFromPunishment;
      tip.appendChild(btn);

      const dismiss = document.createElement('span');
      dismiss.className = 'tip-dismiss';
      dismiss.textContent = '点击泡泡关闭';
      tip.appendChild(dismiss);

      tip.className = 'speech-bubble error-bubble show';
      return;
    }

    renderBoard('practice-board', boards[step + 1], onPracticeClick);
    const m = mistake.punishment[step];
    const tc = m.charCodeAt(2) - 97, tr = 8 - parseInt(m[3]);
    setHighlight('practice-board', tr, tc, true);
    step++;
  }, 800);
}

function revertFromPunishment() {
  if (!punishmentState) return;
  // Restore board to state before wrong move
  practiceState.board = punishmentState.boardBeforeWrong;
  renderBoard('practice-board', practiceState.board, onPracticeClick);
  practiceState.autoPlaying = false;
  practiceState.selected = null;
  punishmentState = null;
  hideAllTips();
  hideFeedback('practice-feedback');
  showFeedback('practice-feedback', '再试一次吧！', 'info');
}
```

- [ ] **Step 4: Update onPracticeClick() wrong-move handler**

In `onPracticeClick()`, replace the wrong-move block (lines 1133-1139):

Old:
```js
    } else {
      // Wrong
      practiceState.streak = 0;
      updatePracticeProgress();
      highlightSquare('practice-board', row, col, 'wrong');
      showFeedback('practice-feedback', '走法不对，再试试！点击棋子重新选择。', 'error');
    }
```

New:
```js
    } else {
      // Wrong move
      practiceState.streak = 0;
      practiceState.selected = null;
      updatePracticeProgress();
      highlightSquare('practice-board', row, col, 'wrong');
      showMistakeFeedback(moveStr);
    }
```

- [ ] **Step 5: Verify error feedback works**

1. Start practice with "意大利开局" (playerSide: white)
2. Play correctly until move 5 (should be Bc4, atMove index 4)
3. Instead of clicking c4, click b5 (f1 to b5) — this is in the mistakes list
4. Verify: red bubble appears with explanation about Bb5 vs Bc4
5. Click "看看后果" — board animates punishment sequence (a6, Ba4, d5)
6. After animation: punchline text + "回到正确位置" button appears
7. Click "回到正确位置" — board reverts, user can retry
8. Make a random wrong move NOT in the mistakes list — verify generic feedback without "看看后果" button

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Add error feedback with consequence demo in practice mode"
```

---

### Task 6: Final integration verification

**Files:**
- No code changes, verification only

- [ ] **Step 1: Full flow test — white side opening**

Open "意大利开局" from the library:
1. Board shows initial position with SVG pieces (white pieces clearly visible on light squares)
2. Black auto-plays after each white move (500ms delay, with highlight and bubble explanation)
3. Play all moves correctly — completion message appears
4. Press left arrow multiple times — board navigates back through moves
5. Press right arrow — board navigates forward, auto-plays opponent if needed

- [ ] **Step 2: Full flow test — black side opening**

Open "西西里防御" from the library:
1. White auto-plays e4 first (500ms delay)
2. User plays c5 as black
3. White auto-plays Nf3
4. Continue through all moves
5. Navigate with arrow keys

- [ ] **Step 3: Error feedback flow test**

Start "后翼弃兵" practice:
1. Play d4 correctly
2. Wait for opponent d5
3. Instead of c4, play Nf3 (wrong move in mistakes list)
4. Red bubble with explanation appears
5. Click "看看后果" — punishment animation plays
6. Click "回到正确位置" — board reverts
7. Now play c4 correctly

- [ ] **Step 4: Verify other modes unaffected**

1. Click "开局百科" — cards display normally
2. Click "开局问答" — quiz works, boards show SVG pieces
3. Click "猜开局名" — guess mode works, boards show SVG pieces
4. Mobile: resize to 400px width — pieces scale, navigation buttons fit

- [ ] **Step 5: Final commit if any fixes needed**

Only if issues were found and fixed in steps 1-4:
```bash
git add index.html
git commit -m "Fix integration issues from final verification"
```

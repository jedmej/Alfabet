# Alfabet - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-file PWA hospital communicator that lets non-verbal patients compose sentences by tapping quick phrases, predicted words, or individual letters.

**Architecture:** Single `index.html` with embedded CSS and JS. App state managed via a plain JS object. Three view modes (phrases, words, alphabet) rendered by building DOM elements programmatically (no innerHTML with user data). PWA via separate `manifest.json` and `sw.js`.

**Tech Stack:** Vanilla HTML/CSS/JS, no dependencies. Service Worker for offline. Optional fetch to LLM API for AI predictions.

**Security note:** All dynamic content uses `textContent` or safe DOM construction. `innerHTML` is only used with hardcoded template strings containing no user input. Dictionary and phrase data is entirely hardcoded.

---

### Task 1: HTML skeleton + CSS layout + mode switching

**Files:**
- Create: `index.html`

**Step 1: Create the full HTML app shell**

Single file with:
- CSS: 3-zone flexbox layout (sentence bar 15vh, content flex:1, tabs 70px), large touch targets (min 60px), high contrast
- HTML: sentence-bar div, content div, word-edit overlay, tab navigation
- JS: App state object, renderSentence(), mode switching, word edit overlay, renderPhrases(), renderWords(), renderAlphabet()

Key CSS details:
- `body`: flex column, height 100%, overflow hidden, font-size 18px, user-select none
- `.btn`: min-height 60px, border-radius 14px, font-size 18px
- `.btn-grid`: CSS grid, 3 columns, 8px gap
- `.alpha-grid`: CSS grid, 6 columns for alphabet layout
- `.word-tile`: styled pills in sentence bar, `.selected` state with purple background
- `.tab`: 70px height, flex column with icon + label
- `touch-action: manipulation` on buttons to prevent double-tap zoom

Key JS details:
- `state` object: `{ sentence: [], mode: 'phrases', selectedWordIdx: null, alphaBuffer: '', activeCategory: 0, editingWordIdx: null }`
- DOM construction uses `document.createElement` + `textContent` for all user/dictionary data
- Tab clicks call `switchMode(mode)` which updates state and calls `renderContent()`
- Sentence bar renders word tiles; tap opens edit overlay; undo removes last word; clear (with confirm) resets
- Word edit overlay: bottom sheet with replacement suggestions from bigrams, "Wpisz inne" button, "Usun" button
- Phrases mode: category pills + phrase button grid; tap phrase sets `state.sentence` to phrase words
- Words mode: shows starters or bigram suggestions; tap adds word; "Wpisz literkami" switches to alphabet
- Alphabet mode: A-Z grid (alphabetical), input display, live-filtered dictionary matches, Polish diacritics toggle, backspace, space/add button
- `addWord(word)`: if `editingWordIdx !== null`, replaces that word; otherwise appends

**Step 2: Verify in browser**

Run: `python3 -m http.server 8000` in the project directory.
Open `http://localhost:8000` on mobile or in Chrome DevTools mobile view.
Expected: 3-zone layout visible, tabs switch between modes, sentence bar shows placeholder text. No data yet (empty phrases/dictionary).

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton with layout, mode switching, and sentence bar"
```

---

### Task 2: Polish dictionary (1000 words + bigrams)

**Files:**
- Create: `slownik.js`
- Modify: `index.html` (add script tag, remove placeholder constants)

**Step 1: Create the dictionary file**

Create `slownik.js` exporting to global scope:
- `PHRASES` object: 5 categories with proper Polish diacritics
  - Potrzeby: "Chce pic", "Chce jesc", "Zimno mi", "Goraco mi", "Chce spac"
  - Bol: "Boli mnie glowa", "Boli mnie brzuch", "Boli mnie reka", "Cos mnie boli"
  - Pomoc: "Zawolaj pielegniarke", "Prosze podniesc lozko", "Prosze polozyc lozko", "Potrzebuje pomocy"
  - Rozmowa: "Jak tam u ciebie?", "Co slychac?", "Dziekuje", "Kocham cie", "Tak", "Nie"
  - Samopoczucie: "Czuje sie dobrze", "Czuje sie zle", "Lepiej mi", "Gorzej mi"
- `DICTIONARY` array: ~1000 most common Polish words, sorted alphabetically, all with proper diacritics
  - Include: common verbs, nouns, adjectives, pronouns, prepositions, conjunctions, question words
  - Hospital-specific: bol, lekarz, pielgniarka, lozko, poduszka, koc, woda, tabletka, zastrzyk, badanie, operacja
  - Time words: dzis, jutro, wczoraj, rano, wieczor, noc, teraz, potem
  - Numbers: jeden, dwa, trzy, cztery, piec...
- `BIGRAMS` object: at least 150 entries mapping word -> array of likely next words
  - "jak" -> ["tam", "sie", "dlugo", "jest", "to"]
  - "chce" -> ["pic", "jesc", "spac", "isc", "wiedziec"]
  - "boli" -> ["mnie", "glowa", "brzuch", "reka", "noga"]
  - "czy" -> ["mozesz", "mozna", "jest", "masz", "bedziesz"]
  - etc.
- `STARTERS` array: common sentence starters ["Ja", "Czy", "Jak", "Chce", "Prosze", "Nie", "Gdzie", "Kiedy", "Co", "To", "Mam", "Jestem"]

ALL text MUST use proper Polish diacritics.

**Step 2: Wire up in index.html**

Add `<script src="slownik.js"></script>` before the main `<script>` block. Remove the placeholder constants from index.html.

**Step 3: Verify in browser**

Reload the app. Expected:
- Phrases tab shows category pills and phrase buttons with Polish diacritics
- Words tab shows starter words; tapping a word shows bigram suggestions
- Alphabet tab filters dictionary words as you type letters

**Step 4: Commit**

```bash
git add slownik.js index.html
git commit -m "feat: add Polish dictionary with 1000 words, bigrams, and hospital phrases"
```

---

### Task 3: PWA setup (manifest + service worker + icons)

**Files:**
- Create: `manifest.json`
- Create: `sw.js`
- Create: `icon-192.svg`
- Create: `icon-512.svg`

**Step 1: Create manifest.json**

```json
{
  "name": "Alfabet - Komunikator",
  "short_name": "Alfabet",
  "description": "Komunikator dla osob ktore nie moga mowic",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#f8fafc",
  "theme_color": "#2563eb",
  "icons": [
    { "src": "icon-192.svg", "sizes": "192x192", "type": "image/svg+xml" },
    { "src": "icon-512.svg", "sizes": "512x512", "type": "image/svg+xml" }
  ]
}
```

**Step 2: Create sw.js - cache-first service worker**

Cache all assets on install: `/`, `index.html`, `slownik.js`, `manifest.json`, icons.
Fetch strategy: cache first, network fallback.

**Step 3: Create SVG icons**

Simple "A" letter in a blue (#2563eb) circle. Two sizes as SVG (scalable).

**Step 4: Verify**

Chrome DevTools > Application > Manifest shows app info. Service Worker registered. "Add to Home Screen" available.

**Step 5: Commit**

```bash
git add manifest.json sw.js icon-192.svg icon-512.svg
git commit -m "feat: add PWA manifest, service worker, and icons for offline support"
```

---

### Task 4: Optional AI prediction

**Files:**
- Modify: `index.html` (add settings UI + AI fetch logic)

**Step 1: Add settings button and panel**

Small gear icon button in sentence bar. Opens a simple settings panel:
- API key text input (saved to localStorage)
- AI toggle on/off
- Close button

**Step 2: Add AI prediction function**

When AI enabled and user picks a word in Words mode:
1. Build prompt with current sentence context
2. `fetch()` to Claude API asking for 9 most likely next Polish words
3. Merge: AI suggestions first, then offline bigram suggestions as fallback
4. Timeout: 2 seconds, then silently fall back to offline
5. API key from localStorage, never hardcoded

**Step 3: Verify**

- Without API key: works exactly as before (offline only)
- With API key: suggestions include AI-predicted words after small delay
- Network failure: graceful fallback, no errors visible to user

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add optional AI word prediction with API key settings"
```

---

### Task 5: Polish, accessibility, and final testing

**Files:**
- Modify: `index.html` (tweaks)

**Step 1: Touch target and spacing audit**

- All buttons min 60px with 8px gaps
- `touch-action: manipulation` on all interactive elements
- Smooth scroll in all modes
- No double-tap zoom issues

**Step 2: Responsive tweaks**

- Tablet/foldable width (>600px): increase grid to 4 columns
- Landscape: adjust layout proportions
- Test sentence bar horizontal scroll with many words

**Step 3: Edge cases**

- Empty sentence: undo/clear are no-ops
- Rapid tapping: no duplicate words added
- Alphabet with no dictionary matches: show "Brak wynikow" message
- Very long sentence: sentence bar scrolls

**Step 4: Commit**

```bash
git add index.html
git commit -m "fix: polish accessibility, touch targets, and edge cases"
```

---

### Task 6: Deploy to GitHub Pages

**Step 1: Create GitHub repo and push**

```bash
gh repo create Alfabet --public --source=. --remote=origin
git push -u origin main
```

**Step 2: Enable GitHub Pages**

```bash
gh api repos/{owner}/Alfabet/pages -X POST -f source.branch=main -f source.path=/
```

**Step 3: Verify deployment**

Open the Pages URL on phone. Add to home screen. Test all modes. Verify offline works after first load.

**Step 4: Fix any path issues if needed and commit**

# iOS Safari Quirks — Required reading before any mobile fix

Living list of iOS Safari / WebKit behaviours that have bitten this codebase.
Each entry: what it is, how to detect, how to fix.

**Consult this BEFORE shipping any mobile-related CSS or layout change.**
Adding a new entry: any time we ship a mobile bug fix whose root cause is
"iOS renders this differently than Chrome/jsdom", add it here.

---

## 1. `<input type="date">` ignores `height` without `appearance: none`

**Symptom**: a `<input type="date">` and sibling `<select>` elements with
identical inline `height: 34px` render at different heights on iOS Safari —
the date input is ~6–8px taller because iOS reserves space for the native
picker chrome.

**First hit**: Wave 1.5 Bug A (2026-05-12) — shipped `height: 34px` thinking
it would equalise. Did nothing on iOS. User reported again as Bug M.

**Fix**:
```jsx
<input
  type="date"
  style={{
    height: '34px',
    WebkitAppearance: 'none',  // ← THIS unlocks the height
    appearance: 'none',
    minHeight: '34px',
    background: 'white',        // appearance:none strips background; restore
  }}
/>
```

**Why**: iOS Safari treats `<input type="date">` with default `-webkit-appearance`
as a controlled-by-OS element. Height, padding, border are all ignored until
appearance is reset. The native picker still opens on focus — `appearance: none`
only affects the inline render.

**Doesn't apply on**: Chrome (Android), Firefox, Edge — all respect `height`
on date inputs directly.

---

## 2. `100vh` includes the iOS address bar

**Symptom**: `min-height: 100vh` on a container makes the page slightly
taller than the visible viewport on iOS Safari, because `100vh` counts the
fully-collapsed address-bar height even when the bar is visible.

**Fix**: use `100dvh` (dynamic viewport height) which excludes the dynamic
chrome:
```css
.full-screen { min-height: 100dvh; }
```

**Fallback for older iOS**: `min-height: 100vh; min-height: 100dvh;` — the
second line is ignored where unsupported.

---

## 3. Hover states stick on touch devices

**Symptom**: a button hovered with finger keeps its hover style after the
finger lifts. Looks like "stuck" highlight.

**Fix**: gate hover styles behind a media query:
```css
@media (hover: hover) {
  .btn:hover { background: var(--primary-blue-hover); }
}
```
Touch devices don't match `(hover: hover)`, so the rule is skipped.

---

## 4. `position: fixed` + notch/safe-area

**Symptom**: a `position: fixed; bottom: 0` element on iPhone X+ overlaps the
home-indicator bar (or gets pushed off by the keyboard).

**Fix**: respect safe-area insets:
```css
.bottom-nav {
  position: fixed;
  bottom: 0;
  padding-bottom: env(safe-area-inset-bottom, 0px);
}
```

For top notch: `padding-top: env(safe-area-inset-top, 0px)`.

For left/right (landscape with notch): symmetric padding —
**both** `safe-area-inset-left` AND `safe-area-inset-right` (asymmetric
padding ships header misalignment — see CLAUDE.md rule #32).

Requires `<meta name="viewport" content="viewport-fit=cover">` in `index.html`
to make safe-area-inset-* return meaningful values.

---

## 5. Date input height differs from select intrinsic height

Related to #1 but worth its own entry because it affects EVERY date input in
a row of mixed input types:

**Symptom**: `<select>` renders at ~32–34px native height on iOS; `<input
type="date">` renders at ~40px. Even with the same padding/font/border, they
stack unevenly.

**Fix**: see #1 — `WebkitAppearance: none` on the date input lets explicit
height match the select.

**Alternative**: bump ALL inputs in the row to 40px (the date's native).
Less surgical but works without appearance reset.

---

## 6. `<button>` font-size doesn't inherit on iOS

**Symptom**: a button inside a card with `font-size: 14px` set on the parent
renders at iOS Safari's default `11px` button font, not 14px.

**Fix**: set `font-size: inherit` (or explicit) on the button:
```css
.btn { font-size: inherit; }
```
Or set explicit `font-size` on every button. Don't rely on inheritance.

---

## 7. Programmatic `<input type="file">` cancellation cannot rely on window.focus on iOS WKWebView

**Symptom**: A click-handler that creates a transient `<input type="file">`,
calls `.click()`, and resolves a promise based on `change` or a `window`
focus-return + setTimeout fallback — silently loses the user's pick on
iOS when the file comes from iCloud Drive. The dropzone never shows the
file and the upload appears not to have happened.

**First hit**: CV-341 / TestFlight Build 22 (2026-05-19). The dropzone in
`CVAnalysisTab.jsx` was migrated from `document.getElementById('cv-file-input').click()`
to `platform.pickFile()`, whose web fallback used:

```js
const onFocus = () => {
  setTimeout(() => {
    if (settled) return;
    settled = true;
    resolve([]);   // ← treated as "user cancelled"
  }, 300);
};
window.addEventListener('focus', onFocus, { once: true });
```

**Why it breaks on iOS**: On iOS WKWebView, when the user dismisses the iCloud
Drive picker, `focus` fires immediately on `window`. The `change` event,
however, only fires after WKWebView materialises the file from iCloud — which
can take >300ms for files that need to download. The setTimeout wins the
race, settles the promise with `[]`, and when `change` finally arrives the
handler bails on `settled === true`. The file is silently discarded; the
calling component sees the same state as if the user had cancelled.

**Doesn't apply on**: jsdom (no real picker), Playwright Chromium (deterministic
event ordering), Capacitor with `@capacitor-community/file-picker` installed
(native plugin bridge bypasses the issue entirely).

**Fix**: Don't use window.focus as a cancellation signal. Two acceptable patterns:

1. **Direct DOM click on a hidden `<input>` with `onChange={handler}`** (preferred
   when a hidden input already exists in the JSX):
   ```jsx
   <div onClick={() => document.getElementById('cv-file-input').click()}>
     {/* dropzone UI */}
     <input id="cv-file-input" type="file" accept=".pdf,.doc,.docx,.txt"
            onChange={handleFileInput} style={{ display: 'none' }} />
   </div>
   ```
   This is what the codebase reverted to in LESSON-094's fix commit.

2. **HTML5 `cancel` event** (for transient-input utilities that need a real
   Promise-returning facade):
   ```js
   input.addEventListener('change', () => resolve(Array.from(input.files)));
   input.addEventListener('cancel', () => resolve([]));  // iOS Safari 16.4+
   ```
   On older browsers without `cancel`, the promise stays pending on user
   cancellation — acceptable because the calling UI keeps its current state
   and the user can re-tap.

**Guard**: `frontend/src/utils/cv-upload-pickfile-gate.contract.test.js` —
forbid-list source-scan that prevents the CV upload click paths from
re-routing through `platform.pickFile()` until `@capacitor-community/file-picker`
lands in package.json. Plus `platform.test.js` "does NOT resolve on window
focus alone" regression guard against re-introducing the heuristic.

---

## Process: before shipping ANY mobile fix

1. Open the changed surface in browser DevTools at 375×667 (iPhone 8 / SE 2nd width)
2. Check this doc for the input/element type you're modifying
3. If your fix touches a known-quirky element, apply the fix pattern documented above
4. Capture a 375px screenshot and attach to the PR (per [PR template](../.github/PULL_REQUEST_TEMPLATE.md))
5. After deploy: open on a real iPhone for visual confirmation (Defect Health Check Q6)

The automated visual regression workflow catches "different from baseline"
post-merge. This doc catches the BEFORE — the known quirks that produce
"different from desktop" in the first place.

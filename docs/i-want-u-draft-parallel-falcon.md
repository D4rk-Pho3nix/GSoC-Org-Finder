# Fix: Scroll-to-top Overlap & Footer Social Icons Wrapping

## Context

Two visual bugs are present on the live site:

1. **Scroll-to-top overlaps the section navigator** — `#back-to-top-btn` (fixed, `bottom:24px; right:24px; z-index:90; 44×44px`) sits at **the exact same pixel position** as `#nextSectionBtn` from the section navigator system (fixed, `bottom:calc(24px + env); right:calc(24px + env); z-index:30; 48×48px`). The scroll-to-top wins the z-index battle and hides the section nav button. On mobile, the `--section-fab-*` variables shift the section nav to `right:12px; bottom:12px` while the scroll-to-top stays at `right:24px; bottom:24px` — the buttons still partially overlap in pixel area. The section indicator pill at `bottom:~152px (desktop) / ~120px (mobile)` is also in the same right-anchored column, making the entire bottom-right zone crowded.

2. **Footer social icons wrap on narrow viewports** — The social links container in `footer.html:111` uses `flex-wrap`, allowing items to overflow onto a new row. On viewports ≤390px (with `px-6` = 24px padding each side → ~342px usable), the combined width of the "Privacy Policy" link + 4×36px icons + gaps is borderline, and X/Twitter drops to its own row.

There is also orphaned CSS: `.scroll-top-btn` and `.scroll-top-btn.visible` (lines 728–750 of `styles.css`) have **no HTML element** in the codebase — dead code from a removed implementation.

---

## Solution

### Fix 1 — Move `#back-to-top-btn` to bottom-left (`src/styles.css` lines 529–566)

The section navigator is permanently anchored at **bottom-right**. There is no gap in the right column to fit the scroll-to-top without overlapping a section nav element (nextSectionBtn, prevSectionBtn, or the pill). Moving the scroll-to-top to **bottom-left** eliminates all conflicts cleanly.

**Changes to `#back-to-top-btn` block (lines 529–566):**

```css
/* BEFORE */
bottom: 24px;
right: 24px;

/* AFTER */
bottom: calc(24px + env(safe-area-inset-bottom));
left: 24px;
/* remove: right: 24px */
```

Add mobile override inside the existing `@media(max-width:640px)` block (line ~607):

```css
#back-to-top-btn {
  left: 12px;
  bottom: calc(12px + env(safe-area-inset-bottom));
}
```

Also remove the hover transform that references vertical movement — it stays the same; no change needed there.

### Fix 2 — Remove dead CSS (`src/styles.css` lines 728–750)

Delete `.scroll-top-btn` and `.scroll-top-btn.visible` rules entirely. No HTML element in the project references this class — confirmed by global search. These are orphaned rules from a previous implementation.

### Fix 3 — Prevent social icons from wrapping (`src/components/footer.html` line 111)

**Change the container div class:**

```html
<!-- BEFORE -->
<div class="flex items-center gap-4 flex-wrap justify-center">

<!-- AFTER -->
<div class="flex items-center gap-3 flex-nowrap justify-center">
```

- `flex-wrap` → `flex-nowrap`: prevents any row wrap; Tailwind `flex-nowrap` sets `flex-wrap: nowrap`
- `gap-4` (16px) → `gap-3` (12px): slightly tighter spacing so all 4 icons + Privacy Policy link fit comfortably at 360px viewports

---

## Files Modified

| File | Lines | Change |
|------|-------|--------|
| `src/styles.css` | 529–566 | Move `#back-to-top-btn` from `right:24px` to `left:24px`, add `env(safe-area-inset-bottom)` |
| `src/styles.css` | ~607 (inside `@media(max-width:640px)`) | Add `#back-to-top-btn { left:12px; bottom:calc(12px + env(safe-area-inset-bottom)); }` |
| `src/styles.css` | 728–750 | Delete dead `.scroll-top-btn` / `.scroll-top-btn.visible` rules |
| `src/components/footer.html` | 111 | `flex-wrap gap-4` → `flex-nowrap gap-3` |

---

## Verification

1. **Desktop (1280px+)**: Scroll down → scroll-to-top appears at **bottom-left**. Section nav FABs + pill appear at **bottom-right** — no overlap. Inspect DevTools to confirm zero pixel intersection.
2. **Mobile 390px (Chrome DevTools)**: Same separation. Footer social icons (Privacy Policy + GitHub + Discord + LinkedIn + X) stay on a single row.
3. **Mobile 360px**: Icons still on one row due to `flex-nowrap`.
4. **No `#nextSectionBtn` hidden**: Toggle section nav visibility — both up/down arrows render without the back-to-top obscuring them.
5. **Safe area (notched device sim)**: Enable `env(safe-area-inset-bottom)` in DevTools → scroll-to-top moves up proportionally, stays clear of the bottom edge.

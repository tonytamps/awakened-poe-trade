# Socket Color Filter Fixes Implementation Plan

> **Execution:** This plan is executed by `bffs:agent-teams` with structured review gates. Do NOT execute tasks directly.

**Goal:** Complete the socket color filter feature by wiring `redSockets`/`greenSockets`/`blueSockets` filters to the trade API and rendering them in the UI.

**Architecture:** Three changes to close the data flow gap: (1) extend the trade API type and query builder to send R/G/B socket filters, (2) add filter buttons to FiltersBlock.vue, (3) add i18n keys. All follow the existing `whiteSockets` pattern exactly.

**Tech Stack:** Vue 3, TypeScript, vue-i18n

---

### Task 1: Add R/G/B socket fields to trade API type and query builder

**Files:**
- Modify: `renderer/src/web/price-check/trade/pathofexile-trade.ts:119-121` (type)
- Modify: `renderer/src/web/price-check/trade/pathofexile-trade.ts:362-364` (query builder)

**Step 1: Add `r`, `g`, `b` to the `sockets` type**

In `pathofexile-trade.ts`, change lines 119-121 from:

```ts
          sockets?: {
            w?: number
          }
```

to:

```ts
          sockets?: {
            w?: number
            r?: number
            g?: number
            b?: number
          }
```

**Step 2: Add propSet calls for red/green/blue socket filters**

In `pathofexile-trade.ts`, after the `whiteSockets` block (lines 362-364), add:

```ts
  if (filters.redSockets && !filters.redSockets.disabled) {
    propSet(query.filters, 'socket_filters.filters.sockets.r', filters.redSockets.value)
  }

  if (filters.greenSockets && !filters.greenSockets.disabled) {
    propSet(query.filters, 'socket_filters.filters.sockets.g', filters.greenSockets.value)
  }

  if (filters.blueSockets && !filters.blueSockets.disabled) {
    propSet(query.filters, 'socket_filters.filters.sockets.b', filters.blueSockets.value)
  }
```

**Step 3: Verify**

Run: `cd renderer && npm run lint`
Expected: No new errors

**Step 4: Commit**

```bash
git add renderer/src/web/price-check/trade/pathofexile-trade.ts
git commit -m "Wire red/green/blue socket filters to trade API query"
```

---

### Task 2: Add i18n translation keys

**Files:**
- Modify: `renderer/public/data/en/app_i18n.json:80`
- Modify: `renderer/public/data/ko/app_i18n.json:77`
- Modify: `renderer/public/data/ru/app_i18n.json:98`

**Step 1: Add keys to English locale**

In `renderer/public/data/en/app_i18n.json`, after line 80 (`"white_sockets": "White: {0}",`), add:

```json
    "red_sockets": "Red: {0}",
    "green_sockets": "Green: {0}",
    "blue_sockets": "Blue: {0}",
```

**Step 2: Add keys to Korean locale**

In `renderer/public/data/ko/app_i18n.json`, after line 77 (`"white_sockets": "흰 홈: {0}",`), add:

```json
    "red_sockets": "Red: {0}",
    "green_sockets": "Green: {0}",
    "blue_sockets": "Blue: {0}",
```

**Step 3: Add keys to Russian locale**

In `renderer/public/data/ru/app_i18n.json`, after line 98 (`"white_sockets": "Белые: {0}",`), add:

```json
    "red_sockets": "Red: {0}",
    "green_sockets": "Green: {0}",
    "blue_sockets": "Blue: {0}",
```

**Step 4: Commit**

```bash
git add renderer/public/data/en/app_i18n.json renderer/public/data/ko/app_i18n.json renderer/public/data/ru/app_i18n.json
git commit -m "Add i18n keys for socket color filters"
```

---

### Task 3: Add filter buttons to FiltersBlock.vue

**Files:**
- Modify: `renderer/src/web/price-check/filters/FiltersBlock.vue:24-25`

**Step 1: Add filter-btn-numeric entries for R/G/B sockets**

In `FiltersBlock.vue`, after lines 24-25 (the `whiteSockets` button):

```vue
      <filter-btn-numeric v-if="filters.whiteSockets"
        :filter="filters.whiteSockets" :name="t('item.white_sockets')" />
```

Add:

```vue
      <filter-btn-numeric v-if="filters.redSockets"
        :filter="filters.redSockets" :name="t('item.red_sockets')" />
      <filter-btn-numeric v-if="filters.greenSockets"
        :filter="filters.greenSockets" :name="t('item.green_sockets')" />
      <filter-btn-numeric v-if="filters.blueSockets"
        :filter="filters.blueSockets" :name="t('item.blue_sockets')" />
```

**Step 2: Verify**

Run: `cd renderer && npm run lint`
Expected: No new errors

**Step 3: Commit**

```bash
git add renderer/src/web/price-check/filters/FiltersBlock.vue
git commit -m "Add socket color filter buttons to price check UI"
```

---

### Task 4: Push and Open Draft PR

**Step 1: Push to remote**

```bash
git push -u fork six-link-color-filter
```

**Step 2: Open a draft PR**

Create a draft PR targeting `Nutschulk/awakened-poe-trade` branch `six-link-color-filter`:

```bash
gh pr create --repo Nutschulk/awakened-poe-trade --head tonytamps:six-link-color-filter --base six-link-color-filter --draft --title "Complete socket color filter feature" --body "<body>"
```

Body should include:
- Summary: what was missing and what was added
- The two code review findings (trade API mapping, UI rendering) as motivation
- Test plan checklist

**Step 3: Report PR URL to user and wait for review**

---

### Task 5: Address Review Feedback

**Step 1: Wait for user review comments**

**Step 2: Address each comment with a dedicated commit. Push after each round of fixes.**

**Step 3: Iterate until the user marks the work as complete.**

---

## Plan Self-Review

### BLOCK 1 -- Plan Summary
- **Goal:** Wire existing redSockets/greenSockets/blueSockets filters to the trade API and render them in the UI
- **Scope:** 3 implementation tasks + 2 completion tasks, modifying 5 files
- **Key decisions:** English placeholder strings for ko/ru locales; follow whiteSockets pattern exactly; PR targets upstream feature branch not master

### BLOCK 2 -- Adversarial Review

1. **Completeness:** APPROVED
   - Trade API mapping (review issue 1) → Task 1
   - UI rendering (review issue 2) → Task 3
   - i18n keys (required by Task 3) → Task 2

2. **Ordering:** APPROVED
   - Task 1 (trade API) is independent
   - Task 2 (i18n) must precede Task 3 (UI uses the keys)
   - Task 3 depends on Task 2
   - Task 4 depends on all implementation tasks

3. **Feasibility:** APPROVED
   - All file paths verified via Read tool in this session
   - Line numbers verified against current HEAD (93c21bf)
   - `fork` remote already configured and tested

4. **Scope:** CONCERN
   - i18n keys use English placeholders for ko/ru. This is pragmatic but the upstream author may want proper translations. Acceptable since we called this out in the PRD as out-of-scope.

5. **Constraints:** APPROVED
   - No test framework exists, so TDD steps are replaced with lint verification
   - Commit frequency: one per logical change (3 implementation commits)

### BLOCK 3 -- Fixes
- CONCERN (scope/i18n): No fix needed. This is a known trade-off documented in the PRD. The PR body will note that ko/ru translations are placeholders.

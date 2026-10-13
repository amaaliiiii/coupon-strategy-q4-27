# Benefit Structure Simplification Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the Benefit Structure panel shorter, clearer, and interactively demonstrate independent versus mutually exclusive Coupons.

**Architecture:** Update the existing inline HTML, CSS, and JavaScript in `index.html`. Preserve the current tab controller and add narrowly scoped state classes for the two revised benefit models.

**Tech Stack:** HTML, CSS, vanilla JavaScript.

## Global Constraints

- Keep the current off-white, black, gray, and coral visual language.
- Do not add dependencies.
- Keep `Unlock & choose` unchanged.
- Avoid horizontal overflow.

---

### Task 1: Compact the panel and revise model interactions

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: Existing `data-subtab-group="benefit"` tab behavior.
- Produces: Independent `.structure-benefit` activation and mutually exclusive `.benefit-option` selection.

- [ ] **Step 1: Simplify markup**

Rename the heading, remove the distinction strip, repeated thesis, comparison cards, and implication block. Add activation buttons to each `Multiple available` Coupon and remove the default selection from `Choose one`.

- [ ] **Step 2: Update visual states**

Add compact active, selected, unavailable, and selection-result styles while preserving the existing responsive grid.

- [ ] **Step 3: Implement interactions**

Bind independent activation to the three `Multiple available` buttons. Bind single selection to `Choose one`, disable competing options after selection, and update the compact result.

- [ ] **Step 4: Verify in browser**

Open the Benefit Structure panel, exercise all activation and selection controls, and confirm no horizontal overflow.

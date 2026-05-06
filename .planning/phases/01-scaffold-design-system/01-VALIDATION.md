---
phase: 1
slug: scaffold-design-system
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-05-06
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — no npm test framework; Shopify CLI + grep |
| **Config file** | None |
| **Quick run command** | `shopify theme check` |
| **Full suite command** | `bash scripts/phase-1-verify.sh` |
| **Estimated runtime** | ~15 seconds (grep-based) |

---

## Sampling Rate

- **After every task commit:** Run `shopify theme check`
- **After every plan wave:** Run `bash scripts/phase-1-verify.sh`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 1-dawn-scaffold | TBD | 1 | SCAFFOLD-01 | Smoke | `shopify theme check` | ✅ after scaffold | ⬜ pending |
| 1-include-tags | TBD | 1 | SCAFFOLD-03 | Grep | `grep -rn "{%[-]*\s*include" --include="*.liquid" . \| wc -l` → 0 | ✅ | ⬜ pending |
| 1-lazysizes | TBD | 1 | SCAFFOLD-03 | Grep | `grep -rn "lazysizes" --include="*.liquid" --include="*.js" . \| wc -l` → 0 | ✅ | ⬜ pending |
| 1-console-log | TBD | 1 | SCAFFOLD-03 | Grep | `grep -rn "console\.log" --include="*.js" . \| wc -l` → 0 | ✅ | ⬜ pending |
| 1-hex-colors | TBD | 1 | SCAFFOLD-03 | Grep | `grep -rn "#[0-9a-fA-F]\{3,6\}" --include="*.liquid" . \| grep -v config/settings_schema.json \| wc -l` → 0 | ✅ | ⬜ pending |
| 1-templates | TBD | 1 | SCAFFOLD-03 | Find | `find templates/ -name "*.liquid" \| wc -l` → 0 | ✅ | ⬜ pending |
| 1-css-tokens | TBD | 2 | SCAFFOLD-02 | Grep | `grep -q "ll-color-bg" layout/theme.liquid && echo OK` | ✅ | ⬜ pending |
| 1-ll-component | TBD | 2 | SCAFFOLD-04 | Grep | `grep -n "customElements.define\|ll:cart:open\|data-ll-" assets/ll-component.js` | ✅ | ⬜ pending |
| 1-no-inline-handlers | TBD | 2 | SCAFFOLD-04 | Grep | `grep -rn "onclick=" --include="*.liquid" . \| wc -l` → 0 | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

None — Phase 1 has no test files to create before execution. All verification is grep-based and runs against the existing filesystem. The phase gate script (`scripts/phase-1-verify.sh`) is created as part of Wave 1 tasks.

*Existing infrastructure covers all phase requirements via CLI + grep.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Theme editor color change updates live preview | SCAFFOLD-02 | No automated way to simulate Shopify theme editor interactions | 1. Run `shopify theme dev`. 2. Open theme editor. 3. Change `--ll-color-bg` value. 4. Verify live preview updates immediately. |
| `shopify theme dev` starts with zero browser console errors | SCAFFOLD-01 | Dev server requires a connected Shopify Partner dev store | 1. Confirm dev store connection. 2. Run `shopify theme dev --store <dev-store-url>`. 3. Open browser. 4. Check console for errors. |

---

## Phase Gate Script

Create as `scripts/phase-1-verify.sh` during Wave 1:

```bash
#!/bin/bash
# Phase 1 artifact audit — all counts must be 0 unless noted
ROOT="."

echo "=== Phase 1 Verification ==="

echo -n "1. {% include %} tags: "
COUNT=$(grep -rn "{%[-]*\s*include" --include="*.liquid" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "2. lazysizes refs: "
COUNT=$(grep -rn "lazysizes" --include="*.liquid" --include="*.js" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "3. console.log calls: "
COUNT=$(grep -rn "console\.log" --include="*.js" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "4. Hardcoded hex in .liquid: "
COUNT=$(grep -rn "#[0-9a-fA-F]\{3,6\}" --include="*.liquid" "$ROOT" | grep -v "config/settings_schema.json" | wc -l | tr -d ' ')
echo "$COUNT (review — focus on sections/ and snippets/)"

echo -n "5. Legacy .liquid templates: "
COUNT=$(find "$ROOT/templates" -name "*.liquid" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "6. ll-component.js exists: "
test -f "$ROOT/assets/ll-component.js" && echo "YES" || echo "NO (must exist)"

echo -n "7. Token injection in theme.liquid: "
grep -q "ll-color-bg" "$ROOT/layout/theme.liquid" && echo "YES" || echo "NO (must exist)"

echo -n "8. customElements.define guard: "
grep -q "customElements.get" "$ROOT/assets/ll-component.js" && echo "YES" || echo "NO"

echo ""
echo "=== Shopify CLI Check ==="
shopify theme check
```

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending

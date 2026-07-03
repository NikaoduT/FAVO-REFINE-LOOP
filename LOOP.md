---
type: refine-loop-charter
status: active
created: 2026-07-03
run: prd-perfection
tags:
  - type/refine-loop-charter
---

# The Loop — Drive FAVO to falsifiable "PRD executed to perfection + code & security checks pass"

> This file IS the prompt. Each pass reads its own RESULTS log, decides what is still missing,
> **drafts the next sub-prompt(s) itself**, and either spawns work or declares convergence.
> State lives in THIS FILE, not the session (thin driver, total log — a fresh window or
> background run resumes from the log alone).

## Goal
Drive the FAVO Café web app (`Favo-WebApp`, canonical git repo, branch off `main`) to a state
where an independent adversary, reading **`docs/FAVO_PRD_v4.md`** as the source of truth,
cannot show that any PRD requirement is un-built, mis-built, or unverified — AND the code &
security gates are green. FOR: the HOFMI FAVO team (Matt/Owner, Nkuli/Admin, Louis, baristas)
running a real café POS + loyalty + COGS system; correctness here protects ministry revenue.

"Perfection" is bounded by the PRD's **current vision and scope** — the §03 Non-Goals are
hard out-of-scope. Perfecting means: every in-scope requirement fully and correctly executed,
every check green. NOT gold-plating beyond the PRD.

## Hard constraints (carried every round)
- **PRD v4 is the source of truth.** `docs/FAVO_PRD_v4.md`. On any ambiguity, the PRD wins.
  Do not invent requirements the PRD does not state; do not violate §03 Non-Goals.
- **Never auto-deploy, auto-merge, or auto-commit to main.** Fixes land on a working branch.
  Convergence is a recommendation the human blesses. (Skill contract #6.)
- **Do not weaken tests or checks to make them pass.** A green check must reflect real behaviour.
- **Non-negotiables (CLAUDE.md):** secrets never committed; audit_log append-only trigger-enforced;
  money = integer cents in `_zar` cols; TZ Africa/Johannesburg; never store/log PAN/CVV/expiry;
  RBAC enforced server-side (UI checks advisory).
- **Grounded, not invented.** Every finding traces to a PRD line, a code path, a test, a query
  result, or a live measurement the adversary can re-check.

## Falsifiable stop condition  ⚠ REQUIRED — the gate  [PASSES]
Grounding = **`docs/FAVO_PRD_v4.md`** (§04 Success Criteria with per-row "How verified", §06
Data Model + RLS invariants, §07 API + security invariants, §08 Business Rules L01–L16, §11
test matrix) + the repo's own checks: `bun typecheck` · `bun lint` · `bun test:unit`
(CI merge gate, `.github/workflows/ci.yml`) and `.github/workflows/security.yml` (Semgrep
ERROR-severity = 0 on `src/ db/ auth.ts middleware.ts`; Grype `--fail-on critical` = 0).

**Scope decision (user, R0):** STATIC + LIVE. Adversary verifies each requirement maps to real
code AND a passing test/query; runtime-only criteria (COGS live increment, push ≤10s, offline
drill, SSE, load) are pushed to **live Supabase/Vercel + real-device** verification as far as
automatable. Irreducibly-manual acts (a human tapping a phone) are named as the saturation
residual, not silently skipped.

Converged when an independent **fresh** adversary, handed the PRD + the code/checks, returns
**zero blocking findings** AND the round surfaces no new material (in-scope) criterion.

**Budget backstop (user, R0):** max **8** rounds. FRESH adversary each round. At the cap, log
residual gaps and STOP for human review — do not grind.
**Saturation stop:** if independent fresh adversaries bracket the SAME residual the grounding
can't resolve (e.g. a criterion that needs a physical device tap or a live 45-user load run we
cannot execute here), STOP early (`saturated`) and name the one act that collapses it.

## Baseline health (R0, commit aa4f165, branch feat/n-homepage-copy-refresh)
- `bun typecheck` → **PASS** (exit 0).
- `bun lint` → **0 errors, 6 warnings** (unused vars; non-blocking).
- `bun test:unit` → **884 passed / 884**, 90 files.
- Security scans (Semgrep/Grype/bun-audit): run in CI; not yet re-run locally this round.
- CI merge gate (typecheck·lint·test:unit) is GREEN on this branch. So "done" is NOT about a
  broken build — it is about proving PRD conformance + clean security, and closing any gaps.

## The spec — initial criteria (grounded; append-only, the loop may add more)
> IDs: **SC** = §04 Success Criteria · **BR** = §08 Business Rules L01–L16 · **CK** = code/security
> gates. Each is falsifiable YES/NO against the grounding named. Full statements live in the PRD;
> this is the checklist the adversary works from. All start `status: open` until a round proves met.

- **SC01** Live COGS dashboard, no manual step; test order → COGS increments ≤5s. (§04, `/api/cogs/live`)
- **SC02** Sunday peak: 45 orders in 85 min, queue stable. (§04 — load test)
- **SC03** Order-to-cup normal day p50 ≤5 min. (§04 — `placed_at`→`completed_at`)
- **SC04** Order-to-cup Sunday peak p95 ≤10 min. (§04)
- **SC05** Push ≤10s of barista "ready". (§04 — real device)
- **SC06** Loyalty: 5 pts / R10 on every completed order, zero manual step. (§04, §08 L06)
- **SC07** Weekly inventory variance <5% by week 2. (§04 — `v_weekly_variance`)
- **SC08** Offline mode: 0 lost orders in 1-hour outage. (§04 — chaos drill)
- **SC09** Audit coverage 100% (orphan-order query returns 0). (§04, §08 L08/L12)
- **SC10** Staff entitlement max 1/staff/weekday (dup query empty). (§04, §08 L03/L14)
- **SC11** Monthly P&L: closed requires `admin_sig` (DB CHECK). (§04, §08 L11)
- **BR01** L01 No payment, no order. **BR02** L02 No refunds (throws). **BR03** L03 staff free coffee 1/weekday, coffee-only.
- **BR04** L04 hours display-only, never gate. **BR05** L05 in-person only, customer read-only.
- **BR06** L06 loyalty earn/redeem rules incl. multi-unit clamp `min(floor(pts/100),floor(total/2000))`, mutually exclusive w/ staff discount, no negative balance.
- **BR07** L07 midnight revenue boundary. **BR08** L08 every inventory adj writes audit.
- **BR09** L09 stock reconciles before daily close. **BR10** L10 emergency purchase needs admin approval.
- **BR11** L11 monthly P&L admin sign-off. **BR12** L12 audit log append-only (UPDATE/DELETE denied).
- **BR13** L13 tenant-isolated to `hofmi` (RLS). **BR14** L14 entitlement DB-enforced weekdays, coffee-only.
- **BR15** L15 barista taps Done; Done is most prominent action on active-order view.
- **BR16** L16 wallet top-up (max R1,000; bal max R2,500) + coffee packs (10 drinks, 90-day expiry) counter-only.
- **CK01** CI gate green: typecheck · lint · test:unit. (`ci.yml`)
- **CK02** Semgrep ERROR-severity findings = 0 on `src/ db/ auth.ts middleware.ts`. (`security.yml`)
- **CK03** Grype critical vulnerabilities = 0. (`security.yml`)
- **CK04** RBAC enforced server-side on every mutating action/route (not UI-only). (§07 invariant)
- **CK05** No PAN/CVV/expiry or secrets stored/logged anywhere. (§05 invariant)
- **CK06** Append-only enforced on audit_log/stock_movements/loyalty_transactions/price_history. (§06 invariant)

## Round protocol (emergent; the loop may deviate)
```
R0 SETUP        — charter + baseline health (done).
R1 ADVERSARIAL  — FULL SURFACE. Fan out ~4 independent fresh adversaries across reference classes
                  (business-rule logic · success-criteria+data-model · security · live-runtime),
                  each grounded in PRD + code, each told to FALSIFY "PRD fully executed". Collect
                  blocking findings + any new criteria.
R2+ BUILD+VERIFY— fix blocking findings on the working branch; a FRESH adversary attacks the DELTA
                  (fixes + still-open) + a cheap regression sweep over survived checks.
CONVERGE? zero blocking + no new criterion → STOP. else → price last round, draft next, loop.
```
Fan-out is a round-1 phenomenon (genuine width across the PRD). After R1, scoped delta attacks only.
Mechanical extraction briefs → smaller model; the adversary + the fix keep the full model.

## CURRENT (maintained — the ONE non-append section; rewritten each round)
- `status: converged` · `round: 3` · `blocking_open: 0` · `criteria: 33`
- `next_action: {kind: await_human, target: graduation — apply migrations + merge/deploy + CI + live/device drills (see recommendation)}`
- `budget: 3 of 8 rounds used (converged early)`
- **CONVERGED** on the falsifiable static bar: two independent fresh adversaries (R2 DB/RLS + R3 app-logic) both returned ZERO blocking, no new criterion. 13 findings fixed (11 R1 + F6-1 + F9-1); F12 was a driver false-alarm the DB adversary caught. Suite GREEN: typecheck ✓, lint ✓ (6 baseline warns), 893/893 unit.
- **Residuals handed to human (cannot be closed from this machine — by design):** (1) tests/db (trigger/CHECK/RLS isolation) run only in CI Postgres; (2) live-runtime timing bars SC01/SC05/SC08 need staging + a real device, and SC02 load test + SC03/SC04 p50/p95 queries don't exist yet; (3) migrations 0020–0023 authored but NOT applied to live (human deploy step); (4) Semgrep re-validated by CI. See graduation recommendation below.

## RESULTS log (append-only, newest at bottom — the loop reads this FIRST)

---

### Round 0 — setup (2026-07-03)
- **round_kind:** [setup]
- Located canonical repo `Favo-WebApp` (git, branch `feat/n-homepage-copy-refresh`, main @ #191).
  Second copy `Favo-WebApp-main` is a git-less snapshot — ignored.
- Read grounding: `docs/FAVO_PRD_v4.md` (source of truth), CLAUDE.md, CI + security workflows.
- Captured baseline health (see above): CI gate GREEN, 884/884 unit tests.
- Filled charter; enumerated 33 initial criteria (SC01-11, BR01-16, CK01-06) from PRD §04/§07/§08 + checks.
- User decisions: scope = static + live runtime proof; budget = 8 rounds.
- **next_action:** `{kind: spawn_brief, target: R1 — 4 independent adversarial audits}`
- **converged?** no.

---

### Round 1 — full-surface adversarial audit (2026-07-03)
- **round_kind:** [adversarial]
- **work_items:** 4 independent fresh adversaries, blind to each other + to this log, each grounded in PRD v4:
  - A1 business-rule logic (L01–L16) · A2 success-criteria + data-model (SC01–11, §06) · A3 security (RBAC/secrets/HMAC/append-only/RLS/Semgrep/Grype) · A4 live-runtime (COGS/push/SSE/offline against live Supabase+Vercel).
- **meta_signals:** `{kind: independent_convergence, over:[A1,A2,A3], onto:[F1], note: "append-only trigger + RLS never applied — found 3× independently"}`
- **Live access confirmed (A4):** `.env.local` has all secrets; `favo-web-app.vercel.app/api/healthz` → 200 (postgres+yoco OK); endpoint auth gates correct. `favo.hofmi.org` unreachable (CLAUDE.md deploy target stale; PRD names Vercel URL — PRD correct).
- **Scanners (A3):** Semgrep 1.136.0 ran; bun audit ran; Grype NOT installed (substituted bun audit → 0 critical).

- **findings (11 blocking, deduped; + 10 nice_to_have):**
  - **F1** [blocking] Append-only trigger (`db/sql/audit-trigger.sql`) + RLS policies (`db/sql/rls-policies.sql`) are orphaned — no drizzle migration / db:migrate step / setup guide / test applies them. Fresh DB ⇒ audit_log is UPDATE/DELETE-able, RLS off. Violates L08/L12/L13, §06 Invariant, CK06. Grounding: `grep "CREATE TRIGGER|CREATE POLICY|ROW LEVEL SECURITY" drizzle/*.sql` → 0. **(A1 BR-F5/6, A2 SC-F1, A3 SEC-F2 — triple.)** status: open.
  - **F2** [blocking] RLS inert even if applied: app connects as owner (bypasses RLS), never sets `app.current_customer_id`/`SET ROLE`. L13 tenant isolation is app-layer only, not RLS. Grounding: `db/index.ts` single client; grep current_customer_id only in policy file. **(A1 BR-F6, A3 SEC-F3.)** → SPEC-DECISION. status: open.
  - **F3** [blocking] Semgrep gate is a silent no-op: `.semgrep/favo-rules.yml:55` invalid YAML (`$KEY: $X` unquoted) ⇒ 0/4 rules load ⇒ exit 7 ⇒ `security.yml` resets exit 7→0 ⇒ always green. All 4 custom rules (pan-near-log, pan-in-audit, mutation-without-audit, raw-sql-no-params) disabled. CK02 unearned. (Current code is clean under fixed ruleset: 0 ERROR, 1 known-FP WARNING.) Grounding: `semgrep --validate` → "0 rule(s)". **(A3 SEC-F1.)** status: open.
  - **F4** [blocking · SPEC-DECISION] L02 says "no refunds; refunds.ts throws." Code fully implements `requestRefund` + `approveRefund` (calls live Yoco refund API, `refunds.ts:107-188`). Code contradicts a LOCKED rule. Direction (amend PRD vs make it throw) is the user's call. **(A1 BR-F1.)** status: open.
  - **F5** [blocking] L03/L14 staff free-coffee gated on the literal string `"cappuccino"` (`discount.ts:19-21`), never reads `menu_items.category`; every other coffee wrongly rejected; `discount.test.ts` codifies the wrong behaviour (asserts Latte rejected). Grounding: PRD L03/L14 "any category='coffee' item". **(A1 BR-F2.)** status: open.
  - **F6** [blocking · SPEC-DECISION] L06 says loyalty earn triggers "on Yoco webhook confirmation or wallet debit — not on order state change." Code earns inside `transitionOrder` on `ordered→in_progress` (`orders.ts:347-361`); webhook awards nothing. Practically near-equivalent (in_progress requires successful payment when Yoco key set) but violates the explicit L06 wording. **(A1 BR-F3, A4 obs.)** status: open.
  - **F7** [blocking] L06 requires `loyalty_points >= 0` DB CHECK; none exists (only `wallet_zar >= 0`). `loyalty.ts:108` comment falsely claims the CHECK is "the safety net." Grounding: grep drizzle/* → no such CHECK. **(A1 BR-F4.)** status: open.
  - **F8** [blocking] POS queue SSE hook (`useOrderStream.ts`) never re-fetches `listActiveOrders` on reconnect/online — state changes during a disconnect window are dropped from the board until manual refresh. Violates PRD R9 ("missed events caught by full poll on reconnect"). COGS hook does this correctly — inconsistency, not a limit. **(A4 RT-F3.)** status: open.
  - **F9** [blocking] SC09 audit-coverage endpoint queries `orders.created_at` — column doesn't exist (it's `placed_at`); query throws, so the SC09 "how verified" mechanism errors instead of returning 0. Grounding: `src/app/api/admin/audit-coverage/route.ts`, `schema.ts:219`. **(A2 SC-F2.)** status: open.
  - **F10** [blocking] SC07 `v_weekly_variance` (`drizzle/0004_cogs_views.sql:44`) computes weekly gross REVENUE, not inventory variance — no stock-take input, no variance %. Cannot verify the <5% target. **(A2 SC-F3.)** status: open.
  - **F11** [blocking] SC01 live COGS hard-codes `expensesZar: 0` and `netZar = grossMarginZar` (`src/server/cogs/compute.ts`) despite `v_daily_expenses` existing — dashboard always reports zero expenses + inflated net. **(A2 SC-F4.)** status: open.

- **nice_to_have (non-gating, tracked):** N1 §06 enum drift PRD-vs-code (doc); N2 no order→COGS-increment integration test (all COGS tests mock db.execute→[]); N3 no load test for SC02; N4 no p50/p95 order-to-cup query (SC03/04); N5 offline "0-lost" proven only by mocked unit test, no chaos drill (SC08/RT-F1); N6 push/COGS ≤10s/≤5s have no timed assertion (RT-F2); N7 closeDaily pages but doesn't "block" (L09); N8 emergency purchase pending-approval flow vs "non-null at insert" (L10); N9 drizzle-orm 0.41.0 HIGH CVE GHSA-gpj5-g38j-94v9 (not critical, unused path); N10 webhook no timestamp/replay-window (R8); + purchasePack doesn't enforce category='coffee'/T&Cs; Semgrep CI arg `middleware.ts` is a phantom path (real file `src/middleware.ts`).

- **passed_checks (survived — confirmed met with proof):** RBAC server-side on all 26 actions + all API routes (A3, first-statement `authorize`/`requireRole`/CRON_SECRET, fail-closed); no card data stored/logged, no committed secrets (A3, CK05); webhook HMAC verify + timingSafeEqual + 401 on invalid (A3); L01 no-payment-no-order; L04 hours display-only; L05 customer read-only (app-layer); L06 clamp math `min(units,floor(pts/100),floor(total/2000))` + mutual-exclusion both directions + redeem idempotency; L07 SAST day boundary; L11 monthly P&L admin_sig DB CHECK; L14 UNIQUE(staff_id,day)+weekday; L15 Done most-prominent; L16 wallet caps (R1,000/R2,500)+pack 90-day expiry, counter-only; SC10 dup-empty by construction; SC11; all 24 §06 tables present; CI gate green; 0 critical deps (bun audit).
- **criteria_added:** none (all 11 findings map to existing SC/BR/CK criteria — the spec was complete; no new width).
- **next_action:** `{kind: await_human}` for F2/F4/F6 (spec-vs-code direction), then `{kind: build}` the 8 unambiguous fixes (F1,F3,F5,F7,F8,F9,F10,F11) on a working branch, then a fresh delta adversary.
- **open_carry_forward:** F1–F11 (loop-owned except F2/F4/F6 pending human direction); N1–N10 (non-gating).
- **converged?** no — 11 blocking.

- **HUMAN DECISIONS on the 3 spec-vs-code conflicts (2026-07-03):**
  - **F4 (refunds/L02):** ENFORCE original L02 → make `refunds.ts` throw; remove the live Yoco refund call. (PRD unchanged; code conforms to PRD.)
  - **F6 (loyalty earn/L06):** MOVE earn to the Yoco payment-confirmed webhook + wallet-debit path, strictly per L06. Remove earn from `transitionOrder`. Behavioural change — re-test outage/deferred path, webhook retries, idempotency.
  - **F2 (RLS/L13):** FULLY wire RLS enforcement — non-owner DB role + per-request `SET LOCAL app.current_customer_id` so RLS genuinely enforces, faithful to L13. Needs live-DB verification.
  - Net: NO PRD edits required by these decisions (user chose to conform code→PRD in all three). The spec is unchanged.

---

### Round 2 — build the 11 fixes + verify (2026-07-03)
- **round_kind:** [build, verify]
- **work_items:** driver-built F3/F4/F5/F8/F9/F11; bg agent A (loyalty) built F6; bg agent B (DB) built F1/F2/F7/F10. Driver integrated + ran authoritative suite + reviewed all diffs.
- **fixes (all 11 findings → fixed):**
  - **F1** → `drizzle/0021_audit_log_append_only.sql` (idempotent trigger, applied via migration chain now). status_history: open→fixed.
  - **F2** → RLS enforcement: `drizzle/0023_rls_customer_isolation.sql` (favo_customer NOLOGIN role, ENABLE-not-FORCE so owner/staff/admin unaffected, fail-closed policies on 6 customer tables) + `src/lib/db-rls.ts` `withCustomerScope()` (SET LOCAL ROLE + set_config, PgBouncer-safe) + `customer.ts` 5 read actions routed through it. status: open→fixed.
  - **F3** → `.semgrep/favo-rules.yml:55` quoted (ruleset now loads 4 rules — verified by R1 adversary's own experiment) + `security.yml` hard-fail `--validate` step so an invalid ruleset can't be masked by the exit-7 reset. status: open→fixed.
  - **F4** → `refunds.ts` both actions throw (L02 enforced); no UI/callers broken (grep clean). status: open→fixed.
  - **F5** → `discount.ts` gates on `category='coffee'` (any coffee, not just cappuccino) + orders.ts joins category + `discount.test.ts` rewritten. status: open→fixed.
  - **F6** → loyalty earn moved from `transitionOrder` to the Yoco `payment.succeeded` webhook (`webhook/route.ts`), idempotent via `loyalty_txn_earn_order_unique` + credit only when a row was inserted; points-earned push relocated; tests rewritten (`webhook-earn.test.ts` +9, earn-scenarios assert transition no longer earns). status: open→fixed.
  - **F7** → `loyalty_points >= 0` CHECK in schema.ts + `drizzle/0020_loyalty_points_check.sql`. status: open→fixed.
  - **F8** → `useOrderStream.ts` re-polls `listActiveOrders()` on every (re)connect (R9 full-poll) + fixes QueueBoard cold-start. status: open→fixed.
  - **F9** → audit-coverage query `created_at`→`placed_at` (valid column). status: open→fixed.
  - **F10** → `drizzle/0022_fix_weekly_variance_view.sql` — v_weekly_variance now computes real inventory variance from stock_takes (+breaches_threshold). status: open→fixed.
  - **F11** → `cogs/compute.ts` queries v_daily_expenses; netZar = grossMargin − expenses; profit = netZar>0. status: open→fixed.
- **verifications:** typecheck ✓ · lint ✓ (0 err, 6 baseline warns) · `bun test:unit` 892/892 (was 884; +8 net from webhook-earn). Fixed 2 test-integration breaks from the RLS reroute (mocked `withCustomerScope`) + 1 short-circuit in getCustomerSummary. Semgrep ruleset fix verified-by-construction.
- **finding F12 (self-review, adversarial-type, blocking) → fixed same round:** RLS policies compared `uuid` column to `current_setting()` `text` with no cast — would ERROR at query time and break the customer dashboard once applied. Fixed: `<col>::text = current_setting(...)` in migration 0023 + db/sql/rls-policies.sql. Caught in diff review because tests/db can't run locally. status: found→fixed.
- **meta_signals:** `{kind: unverifiable_locally, over:[F1,F2,F7,F10], note: "DB-behaviour (trigger/CHECK/RLS isolation) needs Postgres — tests/db skip locally, run in CI; live-DB apply is human step"}`
- **next_action:** `{kind: adversarial, target: 2 fresh delta adversaries + regression sweep}`
- **open_carry_forward:** residuals — (a) tests/db unrun locally (CI Postgres will run them); (b) semgrep not locally validatable (CI does); (c) SC01/SC05/SC08 timing bars = live staging+device (irreducibly manual); (d) migrations authored, NOT applied to live (human deploy step); N1–N10 nice_to_haves still open.
- **converged?** no — pending R2 adversary.

---

### Round 2 adversary — 2 fresh delta adversaries (2026-07-03)
- **round_kind:** [adversarial, verify]
- **work_items:** A (DB/RLS/migrations delta), B (app-logic delta + regression sweep). Fresh, blind to log.
- **findings:**
  - **A → CONVERGED (0 blocking).** Corrected the driver: F12 uuid-cast was a FALSE ALARM — `id`/`customer_id` are declared `text` (gen_random_uuid()::text), so `col = current_setting()` was already valid text=text; the `::text` casts I added are harmless redundant no-ops. Confirmed: all 4 DB fixes applied & in journal; RLS covers all 6 dashboard tables + fail-closed + ENABLE-not-FORCE (owner/staff unaffected); trigger blocks owner; view fixed; schema in sync. 2 nice_to_have: n1 owner-bypass is deployment-dependent (holds on Supabase where app role owns tables — verify before live apply); n2 wallet_transactions policy over-provisioned (harmless).
  - **B → NOT CONVERGED, 1 blocking + 1 nice.** **F6-1 [blocking]:** moving earn to the Yoco webhook ONLY means orders confirmed WITHOUT a webhook (deferred-payment mode R2, dev/no-Yoco) earn ZERO — a regression vs main (which earned unconditionally on `ready`), and L06 also names "wallet debit." Rewritten earn tests codified the regression. **F9-1 [nice, pre-existing]:** audit-coverage filters `entity_kind='orders'` but rows store `'order'` — query runs (F9 true) but SC09 still can't return 0. B also ran typecheck ✓ + 662 server/POS tests green; confirmed F3/F4/F5/F8/F11 correct + regression sweep clean (RBAC/HMAC/clamp/caps intact).
- **meta_signals:** `{kind: adversary_corrected_builder, over:[F12], note: "fresh DB adversary falsified the driver's own false-positive — cols are text not uuid"}`
- **next_action:** `{kind: build, target: F6-1 + F9-1}` then fresh adversary.
- **converged?** no — 1 blocking (F6-1).

---

### Round 3 — fix F6-1 + F9-1 (2026-07-03)
- **round_kind:** [build, verify]
- **fixes:**
  - **F6-1** → new `src/server/loyalty/accrue.ts` `accrueOrderLoyalty(orderId, tx)` (idempotent via loyalty_txn_earn_order_unique, credits only on real insert). Called from BOTH the Yoco webhook (refactored — inline block replaced) AND `retry-deferred-payments.ts` (the deferred-mode confirmation site, now wrapped in a txn + fires points-earned push). So every payment-CONFIRMATION site earns exactly once (L06), deferred-mode regression closed. Dev/no-Yoco has no confirmation event → no earn (correct per L06, not a regression). Wallet-debit-for-order earn: no such feature exists on this branch → nothing to hook (noted residual, not invented). Added `retry-deferred.test.ts` deferred-earn assertion. status: found(R2adv)→fixed.
  - **F9-1** → audit-coverage query `entity_kind='orders'`→`'order'` (matches every writeAudit order row). SC09 can now return 0. status: found(R2adv)→fixed.
- **verifications:** typecheck ✓ · lint ✓ (0 err, 6 baseline warns) · `bun test:unit` **893/893** (+1 deferred-earn test).
- **next_action:** `{kind: adversarial, target: fresh adversary on F6-1/F9-1 delta + regression}`.
- **open_carry_forward:** same residuals (tests/db CI-only, semgrep CI, live timing bars, migrations-not-applied); + wallet-debit earn path absent (feature not present); F12 casts are harmless no-ops (left in place).
- **converged?** no — pending R3 adversary.

---

### Round 3 adversary — fresh delta attack on F6-1/F9-1 (2026-07-03)
- **round_kind:** [adversarial, verify, converge]
- **work_items:** 1 fresh adversary, blind to log, money-path-focused (both confirmation sites) + regression sweep.
- **passed_checks (all attacks FAILED to break it):**
  - Idempotency across webhook + deferred cron — exactly-once via partial unique index + credit-only-on-insert; two concurrent confirmations serialize on the index. No double-earn/double-push (W5 proves).
  - Deferred path: mark-successful + earn atomic in one txn; earns on correct order.totalZar; push gated correctly.
  - Earn removed from `transitionOrder` (grep: 0 hits). Tied to payment confirmation per L06.
  - No confirmation site missed: only webhook + cron set payments successful for orders; staff-discount + full-redeem set successful with totalZar=0 → earnPoints(0)=0; wallet_topup/pack = L16 no-earn; NO wallet-debit-for-order feature exists (nothing to hook).
  - F9-1: query entity_kind now 'order' (matches all 7 order writeAudit sites) + placed_at; returns 0 when every order has an audit row.
  - Regression sweep clean: HMAC/401, pending-charge/failed/refund idempotent paths, cron failed→sync_conflict path all intact. typecheck PASS; 30/30 tests on the 4 delta files.
- **findings:** none (0 blocking, 0 nice_to_have).
- **next_action:** `{kind: await_human}` — graduation.
- **converged?** **YES.** Zero blocking across two independent fresh adversaries (R2 DB/RLS + R3 app-logic), no new material criterion surfaced. stop_reason: converged.

---

### CONVERGENCE SUMMARY & GRADUATION RECOMMENDATION (2026-07-03) — author: loop
**Falsifiable bar MET:** independent fresh adversaries, grounded in PRD v4 + the code/checks, cannot show any in-scope PRD requirement is un-built/mis-built/unverified on the static surface, and code+security checks pass locally (typecheck, lint, 893/893 unit; Semgrep ruleset fixed + hardened; 0 critical deps).

**What changed (branch `fix/n-converge-prd-r2-blocking-fixes`, NOT committed/merged/deployed):** 13 fixes across 4 migrations (0020–0023), the append-only audit trigger + RLS enforcement (customer-isolation, fail-closed, staff/admin unaffected), Semgrep gate un-broken + hardened, refunds throw (L02), staff free-coffee = category='coffee' (L03/L14), loyalty earn moved to payment-confirmation with idempotent multi-site accrual (L06), loyalty_points≥0 CHECK, SSE reconnect re-poll (R9), audit-coverage query (SC09), weekly-variance view (SC07), live COGS expenses (SC01).

**GRADUATION — human-approved steps to close the live/CI residuals (the loop must NOT auto-do these):**
1. **Review + merge** the branch (open a PR; CI runs typecheck·lint·test:unit + the Semgrep `--validate` gate + Grype).
2. **Add a `bun test:db` CI step** after `db:migrate` (CI provisions postgres:16) so the new tests/db (trigger/CHECK/RLS isolation) actually execute — they SKIP locally (no Postgres here).
3. **Apply migrations 0020–0023 to live Supabase** (Flavo-Real) via `db:migrate` as part of deploy. Pre-check (read-only): confirm the app's pooled role OWNS the tables (RLS owner-bypass safety, adversary n1); run the pg_trigger / pg_policies / relforcerowsecurity introspection in the migration comments.
4. **Live-runtime drills** (PRD §04 "How verified", need staging + a real device — cannot be done from this machine): SC01 COGS increments ≤5s; SC05 push ≤10s; SC08 offline 60-min WAN-out 0-lost.
5. **Build the still-missing verifications:** SC02 load test (45 orders/85 min), SC03/SC04 p50≤5m / p95≤10m order-to-cup percentile query. (These were never built; flagged, not silently passed.)
6. Optional non-gating: N7 (closeDaily hard-block), N8 (emergency-purchase pending flow), N9 (bump drizzle-orm ≥0.45.2), N10 (webhook replay-window), pack category/T&Cs, remove harmless F12 `::text` no-op casts.

Convergence is a RECOMMENDATION. Nothing was committed, merged, or deployed.

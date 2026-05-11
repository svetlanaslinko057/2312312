# ATLAS DevOS — Project Deployment

Re-cloned from https://github.com/svetlanaslinko057/575767 (May 11, 2026) and deployed in current Emergent preview pod.

## Phase 1 — Design System Substrate (May-2026)

**Status**: Locked and shipped. Substrate work, no page edits.

**Philosophy**: One operational OS, rendered in two luminance environments
(dark = graphite + sage, light = warm operational paper + deep sage).
NOT a dual aesthetic. Same anatomy, density, typography in both. Only
substrate, contrast, elevation intensity, atmospheric temperature, and
signal saturation differ.

**Architectural decisions locked**:
- signal ≠ success (sage `#8C9B90`/`#4A6B5C` vs olive-shifted `#7E9684`/`#3E5F4F`)
- No marketing palette (emerald/teal/mint/bronze/parchment rejected)
- Status colors de-saturated (ochre warning, oxide danger, slate info)
- Single source of truth: `/app/packages/design-system/tokens/palette.{js,css}`
- Mobile mirror at `/app/frontend/src/design-system/palette.{js,ts}` enforced by `audit/scan_tokens.sh`
- System B (Cognitive Monochrome / parchment+bronze) DELETED — landing now inherits warmth from global light theme
- Tailwind monochrome families (emerald/teal/slate/gray/neutral/zinc) auto-mapped to sage/graphite (no codemod needed for visual healing)
- Semantic palettes (red/yellow/amber/blue/purple/pink) NOT mapped — they keep meaning until Phase 3 review

**Files created**:
- `packages/design-system/tokens/{palette.js, palette.ts, palette.css, semantic.css, spacing.css, typography.css, deprecated.css, spacing.ts, index.css}`
- `packages/design-system/theme/{ThemeEngine.ts, adapter.web.ts, adapter.native.ts}`
- `packages/design-system/typography/index.ts`
- `packages/design-system/motion/{index.ts, motion.css}`
- `packages/design-system/{README.md, index.ts}`
- `audit/PHASE1_TOKEN_MAP.md` (old → new translation table)
- `audit/scan_tokens.sh` (governance — diff check + raw-hex audit + Tailwind class census)

**Files modified**:
- `web/src/index.css` — replaced 290 lines of color declarations with single design-system import
- `web/tailwind.config.js` — hex-direct semantic colors + selective Tailwind monochrome mapping
- `web/src/theme/tokens.js` — reads canonical hex from design-system mirror
- `frontend/src/theme-tokens.ts` — reads canonical hex from design-system mirror (preserves all 93 `T.*` consumers untouched)
- `frontend/src/design-tokens.ts` — became a 60-line shim re-exporting System A via `useTheme()` (was: 110 lines parallel System B with `useColorScheme()` bypass)

**Files NOT touched (intentional, per Phase 1 discipline)**:
- All 96 web pages in `web/src/pages/`
- All ~50 mobile screens in `frontend/app/`
- All web layouts (`AdminLayout`, `ClientLayout`, `DeveloperLayout`, `TesterLayout`)
- Mobile `app/_layout.tsx`, `app/index.tsx`, components

## Phase 1.5 — Stabilization Audit + Fixes (May-2026)

**Status**: 5/5 fixes shipped + re-audit clean.

After Phase 1 substrate work, 12 structural leaks surfaced under real
interface load. Each was traced to a specific file and either fixed
substrate-side or via narrow surgical edits.

**Fixes executed (in locked order)**:

1. **FOUC bootstrap** — `web/public/index.html`. Removed `<html class="dark" style="background:#0B0F14">` hardcode; added inline 8-line `<script>` in `<head>` that reads `localStorage.atlas_theme` → `prefers-color-scheme` → applies `theme-*`/`*`/`data-theme`/`color-scheme`/`background` BEFORE first paint. Measured: `fouc_first_commit` now shows correct theme + bg color from byte 0. Tiny, no React dependency, no imports.

2. **Typography correction** — `web/src/index.css:304`: `body { font-family: 'Inter' }` → `var(--ds-font-body)` (= IBM Plex Sans). `web/public/index.html`: dropped Inter-only `<link rel=stylesheet>`, replaced with preconnect + combined IBM Plex Sans + Space Grotesk + IBM Plex Mono preload. Measured: every probed page now reports `bodyFont: "IBM Plex Sans", -apple-system, ...`. Operational temperature restored.

3. **Mono semantic utility** — `packages/design-system/tokens/primitives.css` adds `.ds-mono` (opt-in: IDs / timestamps / counters / telemetry / currency columns / chain IDs / compact metrics). NOT for hero stats, marketing numbers, descriptive analytics. Plus `.ds-display` for Space Grotesk display headings.

4. **Tailwind text-red/yellow remap** — `primitives.css` overrides `text-red-{300..700}` → `var(--t-danger)` and `text-yellow/amber-{300..700}` → `var(--t-warning)` with `!important` (minimal, intentional — beats Tailwind utility specificity). Backgrounds untouched: `bg-red-*` / `bg-yellow-*` keep loud identity for status badges. **PLUS** patched `web/src/pages/AdminExecutionIntelligence.js:28-54` — replaced 8 inline hex literals (`#ef4444`, `#f59e0b`, `#10b981`, `#06b6d4`) in `sevColor`, `bandStyle`, `verdictStyle` helpers with `var(--t-danger)`, `var(--t-warning)`, `var(--t-success)`, `var(--t-info)`. Cognition surfaces now read observational, not incident-center.

5. **Client gradients removal** — 24 line edits across 6 files (`ClientCabinet.js`, `ClientDashboard.js`, `ClientDashboardOS.js`, `ClientEstimatePage.js`, `ClientHub.js`, `layouts/ClientLayout.js`). Regex-driven: `bg-gradient-to-{r|br} from-{blue|violet|purple|indigo}-{X} to-...` → `bg-signal`; with opacity → `bg-signal/10`; `text-transparent bg-clip-text bg-gradient...` → `text-signal`; `shadow-blue-500/N` → `shadow`; `border-blue-N/M` → `border-signal/30`; `text-blue-{400,500}` (ClientLayout active nav) → `text-signal`. Warmth preserved through paper substrate, sage signal, IBM Plex temperature, whitespace. NOT through gradients.

**Verified by re-audit** (probes + screenshots):
- FOUC: ✅ first commit has theme class + background applied
- Body font: ✅ `"IBM Plex Sans"` reported on every probed page
- Client gradients: ✅ no blue/purple/violet decoration remains on client surfaces
- Cognition severity colors: ✅ "COLLAPSING" headline in `#8E3E3E` (light) / `#B86A6A` (dark) — restrained oxide, not `#EF4444` panic-red
- `audit/scan_tokens.sh`: ✅ CLEAN (mobile mirror synced)

**Architectural decisions locked during Phase 1.5**:
- bg-red-* / bg-yellow-* keep raw Tailwind values (status badges must read alarm).
- text-red-* / text-yellow-* always go through `--t-danger` / `--t-warning` (running text must respect substrate calm).
- `.ds-mono` is opt-in semantic class, NOT inline font-family.
- ClientLayout active nav indicator now uses `bg-signal/10 text-signal` (no decoration).

**Verification**:
- `audit/scan_tokens.sh` → ✅ CLEAN
- Web build → ✅ 496 kB JS, 23.59 kB CSS, no errors
- Expo bundle → ✅ 1555 modules, no resolution errors
- Visual smoke test → ✅ Mobile landing renders deep sage CTA on warm paper (light) / mid sage CTA on graphite (dark) — exactly the master spec.

**Phase 1 known imperfections (Phase 3 backlog)**:
- 68 web pages still contain inline `#hex` strings (mostly gradients on landing)
- 86 mobile screens still contain inline `#hex` strings (e.g. `admin/execution-console.tsx` has full local palette)
- 326 `bg-emerald-*` + 265 `bg-zinc-*` + 48 `bg-slate-*` Tailwind utility usages in web — currently auto-mapped to sage/graphite via tailwind config, but should be migrated to semantic class names

## Phase 1.5b — Dialect breach closure (May-2026, surgical pass)

After Phase 1.5 verification, substrate-grep on the 6 named files surfaced
9 remaining blue-language leaks the previous regex-driven sweep missed.
Per architectural distinction (`inline hex = infrastructure debt` vs
`blue/purple dialect = identity fracture`), these were closed as dialect-
breach, not as Phase 3 codemod.

**9 surgical edits, strict scope (no opportunistic cleanup):**

1. `ClientCabinet.js:53` — Loader2 `text-blue-500` → `text-signal`
2. `ClientCabinet.js:82` — `bg-blue-500/5 blur-[100px]` AI-glow halo **removed entirely** (not recolored to sage glow — the halo motif itself was the leak)
3. `ClientCabinet.js:149` — `bg-blue-500/20` → `bg-signal/15` (next_action review container)
4. `ClientCabinet.js:152` — `text-blue-400` → `text-signal` (Package icon)
5. `ClientCabinet.js:209` — `text-blue-400` → `text-signal` (CheckCircle2 header icon)
6. `ClientCabinet.js:215` — `text-blue-400/60` → `text-signal/60` (CheckCircle2 row icon)
7. `ClientCabinet.js:229` — `text-blue-400` → `text-signal` (under-review counter)
8. `ClientLayout.js:86` — `bg-blue-500/20` → `bg-signal/15` (nav badge)
9. `ClientProjectPage.js:245` + `ClientProjectWorkspaceOS.js:150` — `bg-gradient-to-r from-blue-500 to-{cyan,green}-500` progress bars → **single restrained signal fill** `bg-signal` (operational systems don't need "energetic progress motion")

**Architectural decisions reinforced:**
- Glow halos NEVER recolored — removed. Substrate doesn't need decorative atmosphere; warmth lives in paper luminance, spacing, typography, sage emphasis, silence.
- Progress bars are single-color signal fill. No gradient direction = no implicit "energy flow" reading.
- `bg-signal/15` is the canonical low-pressure sage tint for badges/icon containers (replaces all `bg-blue-500/20` icon-tint instances).

**Verified by post-fix substrate-grep:**
- `ClientCabinet.js` → 0 blue refs (was 7)
- `ClientLayout.js` → 0 blue refs (was 1)
- Listed progress-bar lines in `ClientProjectPage.js:245` and `ClientProjectWorkspaceOS.js:150` → cleaned
- Web rebuild: ✅ 496.64 kB JS, 23.64 kB CSS, no errors

**Still outside surgical scope (NOT touched per "no creep" discipline):**
`ClientProjectPage.js` retains 19 blue refs on lines OTHER than 245
(254, 287, 314, 320, 399, 407, 628, 672, 904, 905…), and
`ClientProjectWorkspaceOS.js` retains 6 (103, 163, 188, 218, 262, 316).
These are same-class dialect leaks. Per architectural rule they should
NOT be deferred to Phase 3 codemod (which targets infrastructure debt
only). Decision required: either expand surgical scope in a 1.5c pass,
or accept them as known-dialect-residue with explicit risk acknowledgment
before Phase 2 opens.

## Access points (current pod)
- Mobile (Expo Web): https://ca52717f-8dc0-4c64-82ad-8435338baff8.preview.emergentagent.com/
- Web platform: https://ca52717f-8dc0-4c64-82ad-8435338baff8.preview.emergentagent.com/api/web-ui/
- Backend API root: https://ca52717f-8dc0-4c64-82ad-8435338baff8.preview.emergentagent.com/api/
- Test accounts: see `/app/memory/test_credentials.md`

## Components

### 1. FastAPI Backend (`/app/backend`)
- 130+ Python modules; main entry `server.py` (~930KB)
- Features: account, escrow, OTP auth, Google auth, Stripe, Cloudinary (mock), team/decision/earnings layers, admin dashboards (v1+v2), legal contracts, AI estimate engine, sentence-transformers (CPU), payment runtime, etc.
- Runs on `0.0.0.0:8001` via supervisor; routes prefixed with `/api`.
- Auto-seeds demo data on startup (admin, dev, client, multi-role users + 2 demo projects, modules, earnings, invoices, deliverables, tickets, notifications).
- Serves the React web build under `/api/web-ui/`.

### 2. Expo Mobile App (`/app/frontend`)
- Expo SDK 54 (expo-router file-based routing).
- Audiences: visitor / client / developer / admin / operator / lead.
- Served on port 3000 (Metro tunnel) → `https://mobile-web-stack-12.preview.emergentagent.com/`.

### 3. React Web Platform (`/app/web`)
- CRA 5 + craco + Tailwind 3 + Radix UI.
- Routes: landing, client/admin/developer/tester auth + dashboards, AdminV2 zones (dashboard, workflow, qa, finance, team, system, payments, profile), client OS, ATLAS DevOS Costs/Operator/Workspace, etc.
- Built once (`yarn build` → `/app/web/build`), served by FastAPI at `/api/web-ui/*`.

## Environment / Integrations
- MongoDB: local (`mongodb://localhost:27017/test_database`).
- LLM: Emergent LLM key wired into `EMERGENT_LLM_KEY`.
- Email (Resend): MOCK (no key set).
- Cloudinary: MOCK (no key set).
- Stripe / Google OAuth / Telegram / push: defaults / inert until keys provided.

## Access points (preview — current)
- Mobile (Expo Web preview): https://web-platform-admin-2.preview.emergentagent.com/
- Web platform landing: https://web-platform-admin-2.preview.emergentagent.com/api/web-ui/
- Admin login: https://web-platform-admin-2.preview.emergentagent.com/api/web-ui/admin/login
- Client auth: https://web-platform-admin-2.preview.emergentagent.com/api/web-ui/client/auth
- Builder (developer/tester) auth: https://web-platform-admin-2.preview.emergentagent.com/api/web-ui/builder/auth
- Backend API root: https://web-platform-admin-2.preview.emergentagent.com/api/

## Test accounts
See `/app/memory/test_credentials.md`.

## Deployment notes
- Disk-pressure mitigation: replaced GPU `torch`/`triton`/`nvidia-*` with CPU-only `torch` to fit /app volume.
- Added `send` package to Expo (peer dep needed by `@expo/cli` ServeStaticMiddleware).
- Web `.env`: `PUBLIC_URL=/api/web-ui` so the bundle picks up correct asset paths.
- Backend `.env` extended with: `EMERGENT_LLM_KEY`, `CORS_ORIGINS`, `APP_URL`, `BACKEND_URL`, `WEB_BUILD_DIR`.

## Causation Propagation (Stage P3.1) — institutional cause-effect chains
A `causation_id` is a single string token that links cognition events belonging to the same institutional cause-effect chain. **No backfill** — propagation begins with new events only. Old records stay `causation_id: null`; their chain is "forming" at read time, which is the honest answer.

**Helper** (`/app/backend/execution_intelligence.py`):
- `ensure_causation_id(db, *, entity_id, root_type, source)` → idempotent. Looks up open chain by `entity_id` (regardless of root_type); reuses if found, otherwise opens new with given root_type. Returns `None` cleanly when entity_id is missing — callers skip propagation rather than crash. Best-effort: failures never block primary writes.

**Storage** (`causation_chains` collection):
```
{causation_id, entity_id, entity_type:"module", root_type, source,
 status:"forming", created_at, last_event_at}
```
P3.1 scope: `entity_type` is always `"module"`. No multi-entity participants. No chain-closing logic yet.

**Write-sites patched (3 only)**:
1. `_detect_and_log_signal_collapse` — opens chain with `root_type="signal_collapse"`, stores `causation_id` on the cognition_event.
2. `_override_create` — joins existing chain or opens with `root_type="operator_override"`, stores id on both the override doc AND its system_actions_log audit entry.
3. The audit-log entry created as side-effect of override inherits the same id (so the chain stays contiguous through the side-effect).

**No other write paths touched.** Suppressions that fire from autonomy loops upstream remain causation-naked in P3.1 — they will join the chain only if/when their entity has already had a signal_collapse logged.

**Reader endpoints**:
- `GET /causal-trace/{causation_id}` → canonical chain reader
- `GET /causal-trace/by-module/{module_id}` → convenience: returns most recent open chain for the module, or `{status:"forming"}` honestly

**Response shape**:
```
{
  causation_id,
  status: "active" | "forming",
  root: {type, entity_id},
  chain: [
    {phase: "signal_collapse",   at, drivers, trigger, source},
    {phase: "suppressed",        at, action_type, source},
    {phase: "operator_override", at, reason, source, ref:{override_id}},
    {phase: "outcome_pending"|"outcome_ai_was_right"|"outcome_operator_was_right"|"outcome_neutral", at, source}
  ],
  interpretation: <one short institutional sentence | "">
}
```

**Interpretation = rule-based, no AI prose**:
- collapse + suppress + override + pending → `"operator challenged a system suppression following cognition collapse; outcome unresolved"`
- collapse + suppress + override + ai_was_right → `"operator overrode a system suppression; suppression was later upheld"`
- collapse + suppress + override + operator_was_right → `"operator overrode a system suppression; operator judgement was vindicated"`
- collapse + suppress, no override → `"system suppression following cognition collapse — no operator intervention"`
- override, no collapse → `"operator override with no upstream system signal"`
- collapse, no suppress → `"cognition collapse detected — no downstream action yet"`

**Forming is correct**: if `<2 distinct phases` are present, response is `{status:"forming", reason:"chain has only one phase so far — propagation continues as new events fire"}` — no fake chain.

**Web surface** (`AdminExecutionIntelligence.js` · `CausalTracePanel`):
- Compact Card between the 3-column operational stream and the Suppression feed. Adjacent to Module Cognition / Timeline so the operator reads "what happened over time" (Timeline) and "what caused what" (Causal Trace) as **complementary** surfaces — not duplicates.
- Horizontal phase row: small rounded cards with top-border colored by phase identity, static `ChevronRight` connectors (no animation, no glow, no orbits).
- Driver chips compressed to `name·severity_letter`, full info in tooltip. Reason quoted, clipped to 2 lines, full text in tooltip.
- Hidden entirely when no module is selected — this is a context surface, not a standalone widget.
- 60s polling parallel to Pattern Memory.

**Acceptance criteria — all met**:
✅ `signal collapse → suppression → operator override → outcome` chain visible when data exists.
✅ `{status:"forming"}` honestly returned when data is insufficient.
✅ No force graph. No global causation map. No AI-generated explanations. No recommendations. No websocket. No historical backfill.

## Multi-Entity Causation (Stage P3.2) — trace augmentation, NOT system modeling
A causation chain can now carry organizational `participants` — the adjacent entities a module's pressure propagated into. **Trace augmentation only.** Not a graph, not a network, not an org map.

**Hard rules baked in:**
- EXPLICIT propagation edges only. If a relationship cannot be explained in one sentence, it is not added.
- NO inference. NO recursive propagation. NO probabilistic edges. NO cross-chain merging. NO "AI influence map".
- Participants are deduped by `(type, id, role)` at write time; idempotent across repeated writes.
- Labels resolved at read time via one batched query — organizational renames stay current; we don't snapshot human-readable labels at write time.

**Allowed edges (P3.2):**
| Edge source | Participant | Role |
|---|---|---|
| Module's `project_id` | project | `origin_pressure_source` |
| Module's `assigned_to` | developer | `assigned_developer` |
| Module's `stack`/`required_skills[0]` | skill_stack | `shared_skill_cluster` |
| `_override_create` operator | developer | `operator` |
| `reassign_task` override target's current assignee | developer | `displaced_assignee` |

**Helpers** (`/app/backend/execution_intelligence.py`):
- `add_participants(db, causation_id, list)` → appends with `(type, id, role)` dedup. Best-effort: failures never block primary writes.
- `_module_participants(db, module_id)` → resolves the project / developer / skill_stack edges for a module.
- `_resolve_participant_labels(db, participants)` → batched read-time label lookup.

**Storage** (extends `causation_chains`):
```
participants: [
  {type: "project"|"developer"|"skill_stack", id, role, added_at}
]
```

**Write-sites patched (same 2 as P3.1 — no new write-sites)**:
1. `_detect_and_log_signal_collapse` → adds module participants when chain is opened/joined.
2. `_override_create` → adds operator, module participants, and (for `reassign_task` targets) the displaced assignee.

`reassignment_recipient` is **deliberately NOT propagated** in P3.2 — capturing it would require a new write-site on the action-execution path, which violates the "stay trace-level" constraint.

**Reader** (`/causal-trace/{causation_id}`): response now includes `participants: [{type, id, role, added_at, label}]`. Forming chains also include participants if any were already attached.

**Web surface** (`CausalTracePanel`):
- New muted section below the chain + interpretation, separated by a dashed top border.
- Kicker: `pressure propagated into` (mono, muted).
- Inline chips: `Project: Mobile App Refresh`, `Developer: John Developer`, `Skill cluster: react_native`. Full `type · role` in tooltip.
- Visual weight intentionally **lower** than the phase chain. Hidden entirely when `participants.length === 0`.
- No expander, no graph, no node explorer, no animation. Same 60s polling as the chain.

**Acceptance criteria — all met**:
✅ Participants surface only when explicit edges exist; otherwise the section is invisible.
✅ Chain remains the primary visual element; participants are secondary trace augmentation.
✅ No new collections, no new background workers, no inference engine, no probabilistic links.
✅ Renames in `projects.name` / `users.name` appear on the next read without backfill.
✅ Verified end-to-end via prod endpoint — `cause_3500ffb02541` returned 4 participants with correct roles & labels.

## Chain Closing Logic (Stage P3.3) — terminal state transition, NOT moral judgment
A causation chain can transition to a terminal state (`closed`) when its decay window has fully passed without entity-side activity. Closure is **rare by design**. Most chains will remain open — that is the honest organizational answer. If a majority of chains close, this layer is producing certainty theatre and has failed.

**Hard rules baked in:**
- Closure is a state transition, NEVER framed as success or failure. None of the 5 states say "AI was correct" or "operator was wrong" — that's Pattern Memory's attribution layer, and this one must not duplicate it.
- Closure requires a **decay window** of `CHAIN_CLOSURE_QUIET_HOURS = 48` hours of no entity-side activity (no new suppressions, overrides, or cognition events on the chain's entity beyond those already part of the chain).
- Closure is **lazy**: evaluated inside `_causal_trace`, persisted idempotently. No new background workers. No new write-sites outside this lazy read path.
- No future predictions. No risk projection. No "likely destabilization". No chain scoring. No recursive closure effects on other chains.
- Conservative-on-failure: when the activity-check query throws, we treat the chain as noisy and refuse to close. Silence must be **proven**, never assumed.
- Premature closure protection: if an override exists but its outcome verdict is `pending` or unknown, closure is refused regardless of decay window.

**Closure states (exactly 5, no synonyms):**
| State | Trigger | What it means |
|---|---|---|
| `pressure_cycle_resolved` | override fired, verdict resolved, module in terminal-completed status | pressure ended after intervention |
| `override_produced_instability` | new signal_collapse / suppressed phase entered the chain AFTER the operator_override | the override didn't quiet the entity |
| `stabilized_without_intervention` | collapse + suppress, no override, 48h of decay | suppression held, no operator action needed |
| `pressure_dissipated` | collapse only, never escalated, 48h of decay | a signal that never grew |
| `outcome_unresolved` | quiet but indeterminate (override resolved without terminal module state, or other unrecognised shape) | chain ended without a clear signal — explicitly NOT a failure |

**Helper** (`/app/backend/execution_intelligence.py`):
- `_entity_activity_after(db, entity_id, after_iso, exclude_causation_id)` → counts institution-visible activity on the entity since `after_iso`, excluding events that already belong to this chain. Any positive count refuses closure.
- `_evaluate_chain_closure(db, chain_doc, phases, override_rows)` → returns `{state, closed_at, decided_because}` or `None`. Pure derivation from phase ordering + module state + override verdict.

**Storage** (extends `causation_chains`):
```
status: "forming" | "closed"
closed_at: ISO timestamp (when closure persisted)
closure: {state, closed_at, decided_because}
```
Once written, closure is idempotent — subsequent reads return it verbatim without re-evaluation.

**Reader response** (`/causal-trace/{causation_id}`):
- Active chain → `{status: "active", chain, participants, interpretation}` as before.
- Closed chain → `{status: "closed", chain, participants, interpretation, closure: {state, closed_at, decided_because}}`. Chain phases continue to render exactly as they did when active — closure is annotation, not replacement.

**Web surface** (`CausalTracePanel`):
- New muted single-line chip at the top of the panel showing the closure label in plain monospace text — `[ pressure dissipated · cognition collapse never escalated ]`.
- Hidden entirely when chain is still active.
- Never colored green/red. Never animated. Never a banner. Never celebrated.
- `decided_because` shown on hover via tooltip.
- `data-testid="causal-trace-closure-{state}"` for each of the 5 states.

**Acceptance criteria — all met**:
✅ Closure NEVER frames an outcome as success or failure.
✅ Decay window enforced — chains < 48h old stay active even if outcome resolved.
✅ Activity inside the quiet window refuses closure — verified via test chain on entity with orphan seed activity (stayed active).
✅ Clean isolated entity → `pressure_dissipated` correctly persisted and read back idempotently.
✅ No new background workers. No new write-sites. No recursive closure effects.
✅ Verified end-to-end via prod endpoint — `cause_test_p33_dissipate` returned `status:closed, closure.state:pressure_dissipated`.

## Calibration Thermometer (Stage P3.C) — a thermometer that does not heal
A single read-only admin endpoint that surfaces RAW structural counts so operators can periodically check whether cognition is drifting into certainty theatre. **Not monitoring. Not governance. Not a dashboard.** A tool for the operator to look at when they choose to look — the system does nothing with the observations itself.

**Strict guardrails baked into the endpoint:**
- NO composite scores. No `cognition_health: 0.73`. No bands. No traffic-light states.
- NO auto-threshold actions. Observations are plain-language strings, never severity-coded, never written anywhere, never emitting events / logs / suppressions.
- NO trend deltas. Single-window snapshot only. No comparisons across runs — that path is monitoring, not calibration.
- NO UI surface. Endpoint only. Operators read it via curl or whatever they prefer, when they prefer.
- NO polling baked in. The system does NOT call this endpoint itself.
- Admin-only (verified — anonymous returns 401).

**Endpoint**: `GET /api/execution-intelligence/calibration?window_days=30` (window clamped to 1–90 days).

**Response shape**:
```
{
  window_days: int,
  closure: {
    total_chains, open, closed,
    by_state: {
      stabilized_without_intervention,
      pressure_cycle_resolved,
      override_produced_instability,
      pressure_dissipated,
      outcome_unresolved
    }
  },
  patterns_raw: {
    suppressed_total,
    suppressed_top_action_type: {action_type, occurrences} | null,
    overrides_total
  },
  silence: {
    modules_total,
    modules_with_cognition_signal_in_window,
    modules_silent_in_window
  },
  observations: [<plain-language strings>],
  generated_at
}
```

**Observation triggers (plain strings, no severity, no action)**:
- `closure density above calibration expectation` — when `closed / total > 0.2`.
- `pressure_dissipated dominates closure distribution` — when `pressure_dissipated > 60%` of closed states.
- `no override_produced_instability instances in window` — when ≥10 chains closed without any landing in this state (override attribution may be too weak).
- `silence baseline diminished — cognition signal density rising` — when fewer than 30% of modules are silent in the window.
- `action_type '<name>' dominates suppression surface` — when one action_type accounts for ≥10 suppressions in the window.

**What the endpoint does NOT do (intentionally):**
- Does not raise alerts. Does not write logs. Does not emit events. Does not change any band. Does not create system_actions. Does not affect closure logic. Does not feed back into Pattern Memory or Topology.
- Does not return any composite score or "health" measure.
- Does not normalize counts into percentages — raw integers only (the operator does their own math if they want).
- Does not compare to previous runs.

**Acceptance criteria — all met**:
✅ Admin-only (anonymous → 401).
✅ Single window snapshot, no trend memory.
✅ Five closure states surfaced as raw counts only.
✅ Observations are plain strings with no severity tag, no associated action.
✅ Endpoint never writes to MongoDB.
✅ Verified end-to-end via prod URL — `/calibration?window_days=30` returned coherent structural counts and four plain-language observations.

## Calibration phase — intentional stillness
At this point the cognition substrate is structurally complete: reasoning surface (P1), continuity (P2.1), override governance (P2.2), institutional memory (P2.3), spatial pressure (P2.4), causation propagation (P3.1), multi-entity participants (P3.2), chain closing (P3.3), and the calibration thermometer (P3.C). The next correct step is **NOT a new layer** — it is observation. Watch:
- Closure distribution over weeks. Does `pressure_dissipated` dominate? Does `override_produced_instability` ever appear? Does the closed-ratio creep above 20%?
- Pattern entropy. Does Pattern Memory start repeating templates? Does silence remain common?
- Semantic drift — the most dangerous signal. If fragments start sounding "smart", if pseudo-poetry emerges, if recognisable cadence forms — cognition is dying into self-reference.

The substrate is finished. No P3.4, no mesh, no chain re-opening. Stillness.

## Pressure Topology (Stage P2.4) — spatial projection of organizational pressure
NOT monitoring dashboard. NOT heatmap. NOT force-graph. Topology is a tertiary cognition layer that answers exactly one question: **where is unresolved organizational pressure accumulating?**

**Default axis: `projects`** (institutional default, governance-centric). Three other axes (`skill_stacks`, `developers`, `action_types`) are reachable in the switcher but return an honest `axis forming` payload — no fake projection invented.

**Endpoint**: `GET /api/execution-intelligence/topology?axis=projects` →
```json
{
  status: "active" | "forming",
  axis: "projects",
  available_axes: [4],
  window_days: 14,
  swimlanes: [
    {band: "high",     label: "HIGH PRESSURE", count, nodes: [...]},
    {band: "elevated", label: "ELEVATED",      count, nodes: [...]},
    {band: "forming",  label: "FORMING",       count, nodes: [...]},
    {band: "quiet",    label: "QUIET",         count, nodes: [...]}
  ],
  total_nodes,
  provenance: {real, replayed, total, replay_share, window_hours},
  generated_at
}
```

**Per-project pressure detectors** (driver, label, severity, occurrences):
- `qa_volatility` — rejected/failed QA decisions
- `revision_cluster` — modules with revision_count ≥ 2
- `override_friction` — cognition_overrides on project's modules (band-agnostic — friction is the signal, not who-was-right)
- `suppression_load` — system_actions_log entries blocked/awaiting
- `overload_risk` — redistribute_load / reassign_task subset of suppression_load

**Band derivation** (`_topology_band`): structural, not numeric.
- `high` — ≥2 high-severity drivers OR (1 high + ≥1 medium)
- `elevated` — 1 high OR ≥2 medium
- `forming` — exactly 1 medium OR ≥2 low
- `quiet` — no contributing pressure

**Severity thresholds tuned for institutional pace** — `high` requires ~10+ events in 14d (not 4+), so most projects under moderate organic load stay in `forming`/`quiet`. Sparse maps are the calm state; full HIGH lanes mean structural distress.

**Web surface** (`AdminPressureTopology.js` at `/admin/pressure-topology`):
- Stacked vertical swimlanes, alphabetical within each band (no "biggest first" — institutional calm).
- Each node: band kicker · project name · `dominant: <driver>` line · severity-dot chips for all contributing drivers (occurrences in tooltip only).
- Empty lanes: `— no projects at this band —` (italic mono, muted). Empty IS the signal.
- 60s polling. Axis switch triggers reload, no animation.
- Provenance subline (one line under title, muted, with `History` icon): `pressure partially derived from replayed cognition traces` — surfaces only when `replay_share ≥ 0.5`. **Tighter than Pattern Memory's ribbon** because topology is tertiary.
- Sidebar entry: `Topology` (Map icon), under Operations group, beneath `Cognition`.

**Guardrails baked in**:
- Band-first, never numbers-first.
- No glow, no pulse, no orbits, no particles, no force-graph.
- No "Suggested action" / auto-recommendations.
- Topology is structurally simpler than Cognition Console — secondary visual weight on purpose.

## Seed Replay (`/app/backend/seed_replay.py`) — transitional cognition tissue
NOT a simulation engine. NOT fake production. Plants temporally-spread RAW events (overrides without fabricated verdicts, QA failures, reassignment waves, overload cascades, suppression clusters) so cognition layer has organic-feeling density before real operator activity accumulates.

**Honesty rules baked in:**
- **No invented outcomes.** Every override is created with `outcome.verdict="pending"` and no `outcome_evaluated_at`. The real `_override_outcome` evaluator fills these only when terminal module signals exist — for replay records that means never, on purpose. Attribution band for replay-only action_types correctly stays `insufficient`.
- **Every record carries provenance**: `source="seed_replay_v1"`, `replay_batch_id`, `replay_generated_at`. Cognition layer computes `replay_share` and shows a panel-level `derived from replayed cognition traces · institutional memory still forming` ribbon when share ≥ 0.5. Never numeric.
- **Idempotent** via `replay_markers` collection. Wipe is explicit and reversible (batch_id → delete_many).
- **Density gradient**: newer days ~2.5x denser than the far edge. No flat, no synthetic pulses.

**Auto-runs on boot** unless `SEED_REPLAY_ON_BOOT=disabled` (label `boot_replay_v1`, 14d, intensity=medium → ~70 events).

**Admin endpoints** (under `/api/execution-intelligence/`):
- `GET /replay/provenance?hours=336` → `{real, replayed, total, replay_share, window_hours}`
- `POST /replay/run` body `{days, intensity, label}` → idempotent on label
- `POST /replay/wipe` body `{label}` → deletes batch by `replay_batch_id`

## Execution Intelligence Console (Stage P1, Web-first)
A cognition surface that turns existing autonomy loops (`assignment_engine`, `auto_guardian`, `module_motion`, `operator_engine`, `event_engine`, `decision_layer`) into a visible, object-centered orchestration view.**Backend** — `/app/backend/execution_intelligence.py` (admin-only):
- `GET /api/execution-intelligence/live-flow` — pipeline + 12-row decision stream
- `GET /api/execution-intelligence/why` — global rationale feed (suppressed + executed)
- `GET /api/execution-intelligence/why/{module_id}` — **structured drivers** for one module: `{driver, label, value, impact (-1..+1), severity}`. Drivers: `overload_risk`, `deadline_asymmetry`, `revision_cluster`, `qa_volatility`, `skill_match`, `confidence_collapse`. Returns verdict (ASSIGNED/IN_FLIGHT/SUPPRESSED/REJECTED/COMPLETED/OPEN) + confidence_band. **Side-effect**: lazily writes `signal_collapse` cognition_event when ≥2 high-severity negative drivers detected (idempotent, 60min dedup).
- `GET /api/execution-intelligence/parallel-universes/{module_id}` — on-demand: Universe A naive (top raw score, no guards) vs Universe B protected (overload/QA/revision penalties applied). Both derived from real assignment_engine signals — no fantasy.
- `GET /api/execution-intelligence/suppressions` — recent suppressed actions with structured drivers (the moat)
- `GET /api/execution-intelligence/conviction` — composite score + **band** `collapsing|weak|building|strong` (no fake quant precision) + signal breakdown
- `GET /api/execution-intelligence/memory` — last 7 days of orchestration outcomes
- **P2.1 — Cognition Continuity**: `GET /api/execution-intelligence/timeline/{module_id}` — temporal reconstruction of reasoning evolution. Hybrid derive from `db.modules` + `db.assignments` + `db.qa_decisions` + `db.system_actions_log` + thin `db.cognition_events` (only `signal_collapse` events). Returns ordered list of `{phase, at, drivers, confidence, source, trigger?, ref?}`. Includes assignment lifecycle, revisions, QA returns, suppressions, rebalances, outcomes.
- **P2.2 — Operator Override**:
  - `POST /api/execution-intelligence/override/{action_id}` — body: `{reason, acknowledged_risk}`. Mandatory reason (min 20 chars) + risk acknowledgement, both validated server-side (422). Snapshots drivers at override-time. Persists to `db.cognition_overrides`. Marks original action as `overridden_by_operator` and writes audit entry to `system_actions_log`.
  - `GET /api/execution-intelligence/override/{override_id}/outcome` — composite outcome evaluator (lazy/delayed): inspects target module terminal state, post-override QA decisions, revision count, duration ratio. Verdict: `operator_was_correct` / `suppression_was_justified` / `neutral` / `pending`. Persists when terminal.
  - `GET /api/execution-intelligence/overrides` — list of overrides for AI Memory attribution chain.
- **P2.3-lite — Pattern Memory** (cross-module organizational memory) — **semantic consolidation pass**:
  - `GET /api/execution-intelligence/patterns` — runs 4 detectors on-demand: `_pattern_overload_suppression`, `_pattern_qa_collapse`, `_pattern_revision_cluster`, `_pattern_operator_override`. Returns list shaped `{pattern_id, category, title, drivers[structured], occurrences, window_days:14, confidence band, attribution {band, ai_was_right, operator_was_right, neutral, pending, total}, scope, representative_module_id, last_seen_at, humility, pressure_rank}` plus `suppressed_count`.
  - **Sort: decision pressure** (`contested` → `operator_dominant` → `ai_dominant` → `insufficient`), NEVER by count. Within band: most recent first.
  - **Hard cap: 7 surfaced patterns** (`MAX_SURFACED_PATTERNS`). Anything beyond surfaces as `suppressed_count` only — UI shows a single muted footer line `+N patterns under threshold`, intentionally non-expandable (Pattern Memory is institutional cognition, not a scrollable list).
  - **Threshold**: ≥3 occurrences in 14d window OR ≥1 with resolved attribution. Otherwise pattern stays in `forming` and is not surfaced.
  - **Attribution band logic**: band requires ≥5 attributed verdicts; 0.7 share → dominant, 0.3+/0.3+ → contested, else `insufficient`.
  - **`_pattern_ai_attribution` retired**: removed entirely. Its signal (AI-vs-operator outcome attribution per `action_type`) is already carried by `_pattern_operator_override.attribution.band` — a separate detector only produced duplicate cards with no new organizational meaning. Removal eliminates analytics-creep and protects semantic gravity.
  - **Contested-aware wording**: `_pattern_operator_override` title becomes `"Operators and AI repeatedly disagree on {action_type}"` when band=contested (live institutional tension), `"Operators override {action_type} repeatedly"` otherwise (mechanical pressure). Wording describes pressure, not metrics.
  - **Per-band humility frame** (the temporal honesty layer): contested → `"organizational signal still contested"`; operator_dominant → `"operators have been overriding consistently · interpretation pending"`; ai_dominant → `"system suppression upheld so far · sample still small"`; insufficient → `"awaiting more outcomes"`; no-attribution patterns (QA / revision) → `"observed across last {14}d"`. No band ever reads as proven truth.

**Web** — `/app/web/src/pages/AdminExecutionIntelligence.js`, route `/admin/execution-intelligence`, sidebar entry "Cognition" (Brain icon):
- 3-column live operational stream (NOT a dashboard grid):
  - LEFT: live module flow (pipeline buckets + clickable stream)
  - CENTER: selected module cognition (assignee strip, confidence band, structured drivers, **continuity timeline ribbon — vertical stepper of phases with colour-coded band dots, "detected" badge for cognition_events, trigger labels**)
  - RIGHT: parallel universes (Universe A naive / Universe B protected)
- Top: cognition band header + 24h pulse (Executed / Suppressed / Awaiting human)
- Bottom: Suppression feed (with **"Operator override" button** per item, filtered to hide already-overridden) + AI memory (with **override-attribution chain rows: "AI suppressed → Operator overrode → Result"** showing reason quote + outcome rationale + signed off-by operator email)
- **Override modal**: suppression context preview (drivers), required reason textarea (min 20 chars, live counter, red border on invalid), risk acknowledgement checkbox, submit disabled until both valid. Submit calls POST and refreshes feeds.
- 15s polling per panel (`POLL_MS`), per-section fetches per `web/ARCHITECTURE.md` ("UI renders JSON, backend is source of truth")
- All `data-testid` populated for testing

**Cognition seed** (`mock_seed.py`) plants 7 `system_actions_log` entries + 2 pending `system_actions` + 6 `qa_decisions` so the console is alive on first render. Idempotent via `mock_seed_markers`.

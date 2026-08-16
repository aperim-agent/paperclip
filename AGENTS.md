<!-- estate-core v2026.08.2 BEGIN — installed by estate realignment; edit via realignment runs only -->
<!-- cspell:words revertable pre-emptively cutoffs cutoff authorisations unauthorised opencode LLAP xhigh behaviour graphify AST -->
# Estate operating rules

## Environment

Autonomous development environment: ephemeral containers, disposable branches, non-production repositories. Every change is version-controlled and revertable. The controls are CI and cross-vendor review, not approval prompts. Prod secrets only via 1Password.

## Standing authorisations (settled — never re-ask)

Permission-bypass flags are correct here and are not to be second-guessed: `claude --dangerously-skip-permissions` · `codex --dangerously-bypass-approvals-and-sandbox` · `opencode --yolo` · `grok` per repo config. You size the work into sensible omnibuses, you pick the model per the ladder below, you merge your own work once the completion bar is met. A run that stops to ask for permission it already has is a failed run.

## Model routing — token discipline is a first-class objective

Route to the cheapest tier that handles the task; escalate only on demonstrated failure, never pre-emptively. Using a frontier model on work a cheaper tier handles is a routing error — it gets fixed in review like any other defect.

- Mechanical (classify, extract, reformat, triage): Haiku 4.5 · GPT-5.6 Luna
- Daily coding (features, refactors, tests, PR review): Sonnet 5 · GLM-5.2 · grok-build-0.1
- Hard code review / independent second author: Grok 4.6 (via LLAP) · GPT-5.6 Terra
- Long-horizon agentic work / multi-agent supervision: Opus 5
- Last resort, after documented failure at the tier above: Fable 5 · GPT-5.6 Sol — one-line justification required in the PR or commit body.

Effort: this repo pins `effortLevel: medium`. Raise it (`--effort high|xhigh`) only for genuinely hard work, and set it at session start — mid-session model or effort switches invalidate the prompt cache. Full table, prices and traps: `docs/agents/model-routing.md` — read it only when actually routing or costing a job.

## Structure and documents — graphify first

`graphify` (local knowledge graph; installed estate-wide) answers structure and navigation from bash under every agent CLI, grok included: `graphify query '<question>' --budget <n>` · `graphify path <A> <B>` · `graphify explain '<node>'`. Query the graph before any grep-and-read sweep for what-defines/calls/uses-what and change-impact questions; fall back to targeted file reads when the graph answer is thin — never to bulk dumps. PDF and Office content is accessed via graphify extraction; raw-dumping binary documents into context is a defect. Keep the graph honest: `graphify hook install` once per clone keeps the code graph current on commit and checkout (AST-only, no LLM cost); document extraction is not covered by the hook — refresh it after adding or changing binary documents, and treat a stale graph as worse than no graph. CLAUDE.md is always exactly `@AGENTS.md`; installers sometimes re-inject guidance there — anything more is drift: reduce it back.

## Testing — proportionality, enforced

Tests prove behaviour changes; they are not progress decoration.

- While iterating, run only the tests targeted at what changed.
- Full suite: at most 2 runs per task (pre-PR gate, post-review fixes). A hook enforces this budget; do not fight the hook — CI runs the suite anyway.
- Never run tests for changes that cannot alter runtime behaviour (docs, comments, non-executed config).
- TDD applies to new behaviour. Chores, docs, config and mechanical refactors need a passing targeted check, not red-green ceremony.

## Context discipline

grep before read; excerpts before whole files; subagents return conclusions, not transcripts. One task per session; compact between phases. Keep this file lean — always-loaded bloat bills on every session and measurably degrades instruction-following.

## Completion bar

Done means all of: full scope implemented · proportional tests exist and pass · PR CI green · cross-vendor adversarial review answered · merged to main. Anything less is reported as unfinished, plainly — reporting partial work as done is the worst available failure mode.

## Obstacles

In order: solve it → route around it (do adjacent work, open a tracking issue) → proceed and record the objection in the PR under `## Concerns`. Stopping is reserved for: destructive operations beyond the disposable environment · the git-irreversible · credentials or customer data · illegal or unauthorised targets. That list does not grow by analogy.
<!-- estate-core v2026.08.2 END -->

## Repository specifics

- Paperclip: control plane for AI-agent companies. V1 build contract: `doc/SPEC-implementation.md`. Read order: `doc/GOAL.md`, `doc/PRODUCT.md`, `doc/SPEC-implementation.md`, `doc/DEVELOPING.md`, `doc/DATABASE.md`.
- Repo map: `server/` (Express API), `ui/` (React + Vite), `packages/db` (Drizzle schema/migrations), `packages/shared` (types/constants/validators), `packages/adapters` (agent adapters), `packages/plugins` (plugin system).
- Dev: `pnpm install && pnpm dev` — embedded PGlite when `DATABASE_URL` is unset. API/UI at `http://localhost:3100`. Reset DB: `rm -rf data/pglite`.
- DB schema changes: edit `packages/db/src/schema/*.ts`, export from `index.ts`, then `pnpm db:generate` and `pnpm -r typecheck`.
- Tests: default is `pnpm test` (Vitest only). `pnpm test:e2e` / `pnpm test:release-smoke` are opt-in browser suites — run only when the change touches them. Pre-hand-off check when scope is broad: `pnpm -r typecheck && pnpm test:run && pnpm build`.
- Domain invariants — enforce, never weaken: every entity is company-scoped with boundary checks in routes/services; single-assignee task model; atomic issue-checkout semantics; approval gates for governed actions; budget hard-stop auto-pause; activity logging on all mutating actions.
- Cross-layer contracts: schema/API behavior changes must stay synced across `packages/db`, `packages/shared`, `server`, `ui`.
- API: base path `/api`; agent access uses bearer `agent_api_keys` (hashed at rest); agents must never access another company's data.
- PRs must fill in every section of `.github/PULL_REQUEST_TEMPLATE.md`, including "Model Used".

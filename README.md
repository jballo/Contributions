# Contribution [1]: Adapter: Moonshot Kimi Model Provider

**Contribution Number:** 1  
**Student:** Jonathan Ballona Sanchez  
**Issue:** https://github.com/orthogonalhq/nous-core/issues/319  
**Status:** Phase I [In Progress]  

---

## Why I Chose This Issue

I want to resolve the issue about adding the missing Moonshot Kimi Model Provider Adapter because I'm interested in using creative ways to use llms and benchmarking them to see the practical use of them. I have experience with llms, rag, and llm evals which is why I think this issue is a strong fit.

Why it's a good match:
1. **Relevant experience** — I've worked directly with provider APIs and their request/response formats.
2. **Contained scope** — it follows an existing adapter pattern instead of
   touching core logic, making it a realistic contribution.
3. **Clear gap** — nous-core supports other providers but can't call Kimi yet,
   so the goal is well defined.

What I'll get out of it most is adapting to someone else's architecture and
conventions rather than my own.


---

## Understanding the Issue

### Problem Description

The nouse application does not currently support the Moonshot Kimi provider. Users cannot route chat or agent work to Kimi models through Nous's model routing system.

### Expected Behavior

A new leafe is expecte let users:
- Register Moonshot Kimi as a selectable remote provider
- Invoke and stream Kimi models via `IModelProvider`
- Validate requests against `TextModelProvicer`
- Store/user a Kimi API key through the existing credential vault flow
- Appear in generated provider catalogs and pass adapter tests

### Current Behavior
- Moonshot Kimi is not implemented. Only certified leaves exist for Anthropic, OpenAI, and Ollama under `self/subcortex/providers/src/providers/`. Selecting or assigning a  Kimi model has no support provider path.

### Affected Components
- New provider leave: `self/subcortex/providers/src/providers/moonshot-kimi/` which will include definiton.ts, adapter.ts, provider.ts, index.ts (reusing openai api protocol if comptaible)
- Contract: `self/shared/src/interfaces/subcortex.ts` (`IModelProvider`, `TextModelInputSchema`)
- Generated catalogs: `provider-definitons.ts`, `provider-adapters.ts`, `provider-factories.ts`
- Tests: `self/subcortex/providers/src/__tests__`
- Config/UI (downstream): credential vault + model picker once provider is registered

---

## Reproduction Process

### Environment Setup

### Steps to Reproduce

#### Prerequesities (environment)
1. Install and wire nvm (macos users may use homebrew)
```bash
brew install nvm
mkdir -p ~/.nvm
cat << 'EOF' >> ~/.zshrc
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"
EOF
source ~/.zshrc
```
2. Install pnpm and open new terminal subsequently
```bash
brew install pnpm
```
3. Install Node 22
```bash
nvm install 22
nvm use 22
```
#### Repo Setup
1. Forked `orthonalhq/nous-core` 
2. Clone forked repo locally and add upstream
```bash
git clone git@github.com:jballo/nous-core.git
cd nous-core
git remote add upstream https://github.com/orthogonalhq/nous-core.git
```
3. Fetch upstream and create feature branch from integration branch (not main):
```bash
git fetch upstream
git checkout -b feat/moonshot-kimi-provider upstream/feat/contributor-friendly-inference-provider-surface
```
4. Install and verify
```bash
pnpm install
pnpm build
pnpm test
```

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/jballo/nous-core/commit/82120683a7a776c9cc4017ac289b259386421983
- **Screenshots/logs:**
   ![pnpm build success](https://ix7l8rtzlf.ufs.sh/f/fsfgZfmadWBrDCg0PZeqxcUSLEYafZiTVsB4t7Fdw8poAuQN)
   ![pnpm test success](https://ix7l8rtzlf.ufs.sh/f/fsfgZfmadWBrE7q45LxNh8LVbpMHZqUJ4oI7jsWfzX6SmClA)
- **My findings:** Synced to `feat/contributor-friendly-inference-provider-surface` on branch `feat/moonshot-kimiprovider`. After `pnpm install`, build and tests passed. Provider leaf layout is in place (`anthropic/`, `ollama/`, `openai/`). Ready to implement Moonshot Kimi leaf.

---

## Solution Approach

### Analysis

The gap is a missing certified provider leaf — not a runtime or resolver bug. Nous only has leaves for Anthropic, OpenAI, and Ollama under `self/subcortex/providers/src/providers/`, so Moonshot Kimi never lands in the generated catalogs (`provider-definitions.ts`, `provider-adapters.ts`, `provider-factories.ts`) and there is no path to invoke Kimi through `IModelProvider`. Root cause: no `moonshot-kimi/` leaf or contracts yet.

### Proposed Solution

Add a certified `moonshot-kimi` leaf under `self/subcortex/providers/src/providers/moonshot-kimi/` with `definition.ts`, `adapter.ts`, `provider.ts`, and `index.ts`. Kimi's API is OpenAI Chat Completions–compatible, so I'll reuse `src/protocols/openai-api/` for shared wire logic instead of building a new protocol. After the leaf is in place, regenerate catalogs — no hand-editing generated files or `adapter-resolver.ts`.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Nous cannot talk to Moonshot Kimi because there is no provider leaf. Users can't select Kimi models, stream responses, or store a Kimi API key through the vault.

**Match:** Closest pattern is OpenAI Chat Completions → `src/protocols/openai-api/` + `src/providers/openai/`. I'll read contracts in `src/schemas/` before changing anything.

**Plan:**
1. Reuse `openai-api` protocol; only add a new protocol module if testing shows Kimi-specific wire differences.
2. Create `self/subcortex/providers/src/providers/moonshot-kimi/` with `definition.ts`, `adapter.ts`, `provider.ts`, `index.ts` (skip `implementation.ts` if shared OpenAI protocol handles execution).
3. Fill contracts: `providerDefinition` (`as const satisfies ProviderDefinition`), `providerAdapter`, `providerFactory` (`as const satisfies ProviderFactoryModule`). Keep definitions metadata-only — no env reads, network, or instantiation.
4. Set `vendorKey`, `adapterKey`, and `protocol` correctly; declare auth with env var + vault namespace for this remote provider.
5. Regenerate catalogs: `pnpm --filter @nous/subcortex-providers run generate:providers` then `check:generated`.
6. Add tests per the [Testing Checklist](https://docs.nue.orthg.nl/docs/development/provider-adapters/testing-checklist): definition, codegen, adapter (`formatRequest` / `parseResponse`), implementation (errors, streaming, credentials), runtime resolution.
7. Run focused checks: `typecheck` + targeted `vitest`.

**Implement:** Working on branch `feat/moonshot-kimi-provider` —> will link commits here as the leaf lands.

**Review:**
- [ ] Changes stay in the provider leaf (or shared `protocols/`), not generated catalogs or runtime vendor branches
- [ ] Did not hand-edit `provider-definitions.ts`, `provider-adapters.ts`, or `provider-factories.ts`
- [ ] Did not edit `adapter-resolver.ts` unless resolution policy actually changed
- [ ] Definition is metadata-only; adapter translates; implementation executes; factory constructs
- [ ] Tests use Moonshot-specific fixtures, not copied Anthropic shapes
- [ ] Matches CONTRIBUTING.md provider adapter expectations

**Evaluate:**
```bash
pnpm --filter @nous/subcortex-providers run generate:providers
pnpm --filter @nous/subcortex-providers run check:generated
pnpm --filter @nous/subcortex-providers run typecheck
pnpm --filter @nous/subcortex-providers exec vitest run \
  src/__tests__/provider-codegen.test.ts \
  src/__tests__/public-exports.test.ts \
  src/__tests__/provider-definitions \
  src/__tests__/adapter-resolver.test.ts \
  src/__tests__/provider-pipeline-integration.test.ts \
  --config vitest.config.ts
```

---

## Testing Strategy

### Unit Tests

**Added** `moonshot.test.ts` — a new dedicated suite for the Moonshot Kimi provider leaf:


- [x] Definition: asserts the canonical alias, that no built in provider id is hand authored, the identity/protocol/credential metadata (`vedorKey: moonshot`, `chat-completion` protocol + adapter, `https://api.moonshot.ai`, `kimi-k2.6`, `MOONSHOT_API_KEY` auth), hydration into `PROVIDER_DEFINITIONS` with a derived built-in id, and `ProviderDefinitionSchema` validation
- [x] Adapter: reuseds the shared `chat-completion` adapter, parses a Kimi text response, parses Kimi native tool calls, and returns a text fallback (no throw) on malinformed output
- [x] Factory: registered under the `moonshot` vendor key and constructs a `ChatCompletionsProvider` from the supplied credential

### Integration Tests

**Updated** existing catalog/pipeline suites to include `moonshot` (no behavior of other providers changed):

- [x] `provider-pipeline-integration.test.ts`: added `moonshot` to the ordered vendor catalog, asserted it resolves to the `chat-completions` adapter, that it constructs a `ChatCompletionsProvider` from a `MOONSHOT_API_KEY` env credential, and it's absent from lane aware registry lookups (plus env-var cleanup in `afterEach`)
- [x] `adapter-resolver.test.ts`: added the `moonshot` vecor -> `chat-completions` adapter key assertion (and catalog ordering entry)
- [x] `provider-codegen.tests.ts`: added `moonshot` to the expected aggregate-codegen output, keeping generated files in sync
- [x] `provider-definition-types.test.ts`: updated to account for the new leaf

### Manual Testing

Ran the affected suites locally with `vitest run` (moonshot, pipeline-integration, adapter-resolver, provider-definitions, provider-codegen): **5 files, 38 tests passing**.

**Added** a gated live blackbox suite `moonshot.live.test.ts` that hits the real Moonshot Kimi API for both a chat completion and a streamed response. It is `it.skip` by default (never runs in CI, needs no credential). Run manually with:

```bash
NOUS_MOONSHOT_LIVE_BT=1 MOONSHOT_API_KEY=sk-... \
  pnpm --filter @nous/subcortex-providers exec vitest run src/__tests__/providers/moonshot.live.test.ts
```

---

## Implementation Notes

### Week 1 Progress

Built the Moonshot Kimi provider leaf end-to-end against the current provider-leaf contract from #319, including tests and live API validation. The key realization driving the design was that Moonshot's API is OpenAI compatible chat-completions, so the leaf is **pure metadata + catalog wiring** — there is no bespoke protocol code.

**What I built:**
- Authored the leaf as a `ProviderDefinitionLeaf` (`definition.ts`): `vendorKey: 'moonshot'`, `protocol`/`adapterKey: 'chat-completions'`, endpoint `https://api.moonshot.ai`, default model `kimi-k2.6`, `MOONSHOT_API_KEY` auth (`api_key`, required, `moonshot` vault namespace), and `capabilities: { streaming: true, nativeToolUse: true }`
- Deliberately did **not** hand-author a `wellKnownProviderId` — the built-in id is derived from `vendorKey` via `deriveBuiltInProviderId('moonshot')` during hydration into `PROVIDER_DEFINITIONS`, which a test asserts explicitly
- `adapter.ts` re-exports the shared `chatCompletionsAdapter` from `protocols/openai-api/adapter.ts`; `provider.ts` is a thin `ProviderFactoryModule` that constructs a `ChatCompletionsProvider` from the supplied credential. No `implementation.ts` was needed.
- Registered the leaf in all three certified catalogs (`provider-definitions.ts`, `provider-adapters.ts`, `provider-factories.ts`) and updated registry wide expectations (catalog ordering, adapter resolution, pipeline construction, codegen aggregate)
- Wrote the dedicated unit suite `moonshot.test.ts` (definition / adapter / factory) and a gated live blackbox suite `moonshot.live.test.ts` modeled on `codex-cli.live.test.ts`

**Challenges / decisions:**
- The model originally referenced (`kimi-k2-0711-preview`) was deprecated by Moonshot on 2026-05-25 and now 404s from the live API. I caught this only by running the gated live test against the real endpoint. Updated the default model to `kimi-k2.6` and fixed the affected expectations in `definition.ts`, `moonshot.test.ts`, and `provider-definitions.test.ts`
- Confirmed the gated live suite passes (invoke + stream, 2 passed) and re-ran all affected local suites

### Code Changes

**Files added (leaf):**
- `self/subcortex/providers/src/providers/moonshot/definition.ts`
- `self/subcortex/providers/src/providers/moonshot/adapter.ts`
- `self/subcortex/providers/src/providers/moonshot/provider.ts`
- `self/subcortex/providers/src/providers/moonshot/index.ts`

**Files added (tests):**
- `self/subcortex/providers/src/__tests__/providers/moonshot.test.ts`
- `self/subcortex/providers/src/__tests__/providers/moonshot.live.test.ts`

**Files modified (catalogs):**
- `self/subcortex/providers/src/provider-definitions.ts`
- `self/subcortex/providers/src/provider-adapters.ts`
- `self/subcortex/providers/src/provider-factories.ts`

**Files modified (registry-wide test expectations):**
- `self/subcortex/providers/src/__tests__/adapter-resolver.test.ts`
- `self/subcortex/providers/src/__tests__/provider-pipeline-integration.test.ts`
- `self/subcortex/providers/src/__tests__/provider-definitions/provider-definitions.test.ts`
- `self/subcortex/providers/src/__tests__/provider-codegen.test.ts`
- `self/subcortex/providers/src/__tests__/provider-definition-types.test.ts`

**Key commits:**
- `32c78078` — feat(providers): add moonshot kimi provider leaf and register in catalogs
- `26b89226` — test(providers): cover moonshot leaf definition, adapter, and factory
- `7a217944` — fix(providers): update moonshot default model to kimi-k2.6
- `d955741d` — test(providers): add gated live test for moonshot kimi provider

**Approach decisions:**
- **Reuse the `openai-api` protocol instead of writing new code.** Moonshot is OpenAI chat-completions compatible, so the leaf reuses the shared adapter and `ChatCompletionsProvider`. This keeps the new code surface to metadata + wiring and inherits existing text/tool-call/streaming parsing behavior for free
- **Derive the built-in id rather than hand-author it.** Following the current #319 contract, `wellKnownProviderId` is derived from `vendorKey` at hydration time, so the leaf carries no hardcoded id.
- **Gate the live test, never run it in CI.** `moonshot.live.test.ts` is `it.skip` by default and only enabled with `NOUS_MOONSHOT_LIVE_BT=1`, so CI needs no credential and stays deterministic, while still giving a real-API smoke path for both invoke and stream
- **Default to `kimi-k2.6`.** Chosen after the previous `kimi-k2-0711-preview` default was found deprecated (404) against the live API

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- **Jun 7, 2026** ([issue #319](https://github.com/orthogonalhq/nous-core/issues/319)): `main` is a release branch — don't develop there. Fork off `dev` or a team integration branch. Adapter surface refactor coming on a special integration branch for the class.
- **Jun 11, 2026:** Target `feat/contributor-friendly-inference-provider-surface` for PRs, not `dev`. Implement as certified provider leaves under `self/subcortex/providers/src/providers/<vendor>/`; don't hand-edit generated catalogs. [Provider adapter docs](https://docs.nue.orthg.nl/docs/development/provider-adapters) are the source of truth.
- **Jun 13, 2026:** Fast-forwarded the integration branch after I flagged it was still on the flat adapter layout while the docs described the leaf structure.

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

Before I could reproduce the gap locally, I hit a few blockers from the [issue thread](https://github.com/orthogonalhq/nous-core/issues/319):

1. **Wrong starting branch** — My fork defaulted to `main`, which doesn't have the contributor-friendly provider surface. On `main`, `OpenAiCompatibleProvider` still hard-codes `OPENAI_API_KEY`, so Kimi can't ship as a simple config entry until the shared chat-completions refactor ([#324](https://github.com/orthogonalhq/nous-core/issues/324)).
2. **Stale issue body vs current approach** — The issue pointed at a flat `moonshot-kimi-provider.ts` file. The maintainer clarified Kimi should be a **certified provider leaf**, not a standalone adapter class.
3. **Integration branch lag** — `feat/contributor-friendly-inference-provider-surface` was briefly behind the docs (flat `src/adapters/` layout vs `src/providers/<vendor>/` leaves). I asked in the thread; the maintainer fast-forwarded the branch.

**How I resolved it:** Confirmed branch and approach in the issue thread before writing code, then based off `feat/contributor-friendly-inference-provider-surface` (see Repo Setup). Build and tests passed once synced.

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Issue #319 — Moonshot Kimi Model Provider](https://github.com/orthogonalhq/nous-core/issues/319) (setup blockers and maintainer guidance)
- [Provider adapter docs](https://docs.nue.orthg.nl/docs/development/provider-adapters)
- [Tutorial or Stack Overflow post that helped]

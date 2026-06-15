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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

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

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

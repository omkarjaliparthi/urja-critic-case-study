# 06 · Build vs Buy

## The question

In May 2026 there is a real eval-tool market. LangSmith, Braintrust, OpenAI Evals, Anthropic Evals, Galileo, Arize, Weights & Biases — all ship eval orchestration with judge-model support. So why build a Critic at all?

The short answer: **the Critic is not a competitor on orchestration.** It is a *policy layer* — cross-provider grading + public-domain reference corpus + structured rubric grammar + actionable failings — that runs *on top of* any orchestration. Build vs buy is the wrong frame. Most teams should *also buy* an orchestrator. The Critic's job is the policy guarantee that orchestrators do not provide.

## The landscape

| Tool | Strength | What it does NOT do |
|---|---|---|
| **LangSmith / LangChain Eval** | Eval orchestration, dataset management, run tracking, integration into the LangChain SDK | Cross-provider judging is opt-in glue; reference-corpus provenance is the user's problem |
| **Braintrust** | Polished UI for eval runs, prompt-comparison playground, dataset versioning | Same — single-provider judging is the default; no policy enforcement on judge identity |
| **OpenAI Evals** | First-party harness for OpenAI models, large built-in eval suite | Locked to OpenAI judges. Cannot grade GPT-generated output with non-GPT judges in the supported path |
| **Anthropic Evals** (developer docs) | First-party harness for Anthropic models, developer-friendly | Locked to Anthropic judges. Same limitation, opposite direction |
| **Galileo / Arize / W&B** | Production observability for LLM apps, drift monitoring, hallucination detection | Different problem (monitoring, not gating). Useful sidecar; not a release gate |
| **Naive LLM-as-judge** (DIY) | Cheapest, simplest, ships in a day | Same-provider self-preference bias documented since 2023 |
| **Human-only review** | No bias from same-provider grading; high-fidelity feedback | Doesn't scale beyond a handful of artifacts per day |
| **Urja Critic** | Cross-provider by design, public-domain reference corpus, rubric-grammar policy layer | Not an orchestrator. Uses any orchestrator's runner. Not a dataset manager. |

## The four real options I considered

### Option 1 · Use LangSmith / Braintrust + write the policy myself

Cost: ~1 day of integration work + ongoing tool subscription.

Outcome: The cross-provider rule lives in *my* glue code, not the harness. Every developer who picks up the codebase has to know to enforce it. Defaults are single-provider; the wrong configuration is one PR away. The policy is procedural, not structural.

The sentiment that killed this: *"if the policy lives in the README, the policy doesn't live."*

### Option 2 · Use OpenAI Evals or Anthropic Evals

Cost: free tooling, locked to one provider for judging.

Outcome: Provider-locked judging is exactly the failure mode the Critic exists to prevent. The structural cross-provider guarantee cannot be retrofitted onto a first-party harness whose entire premise is "we grade our own family."

This option is correct for teams that don't care about same-family bias. The Critic exists for teams that do.

### Option 3 · Build a Critic from scratch

Cost: ~2 weeks of engine + adapter work, plus ongoing maintenance of two provider adapters.

Outcome: The policy is the product. Cross-provider non-identity, anchored-scale rubrics, public-domain reference corpus, structured failings — all encoded into the type system, all enforced at the boundary. Every aspect of the eval gate is a deliberate choice, not a default inherited from a vendor.

Maintenance is two adapters today (OpenAI today, Claude adapter is a sibling file behind the same interface) — small surface area. The cost compounds slowly because the rubric grammar absorbs new domains without engine changes.

### Option 4 · Hire a vendor for "managed eval"

Cost: $X / month, opaque pricing, opaque grading methodology.

Outcome: The customer-legal conversation moves from *"why this score?"* to *"why does the vendor say this score?"* — which is worse, not better. Defensibility requires explainability, and explainability requires owning the grading methodology.

For my surface (consumer SaaS, paying users, customer-legal exposure), the vendor-managed path was a non-starter.

## Why Option 3 was the right call for this surface

Three reasons specific to the Insights by Omkar context:

1. **The cross-provider policy is the product.** Outsourcing the policy to a vendor would be outsourcing the case-study hook to a vendor.
2. **The reference-corpus provenance has to be ours.** A managed vendor whose judge cites in-copyright modern textbooks would re-introduce the copyright exposure the [public-domain corpus decision](./05-decision-public-domain-corpus.md) was built to eliminate.
3. **The architecture generalizes across the rest of the API ecosystem.** The same scoped-JWT auth, dual-backend rate-limit, and adapter-pattern interface used by [Kriya](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study) and Urja's other tracks. Building a Critic that fits the existing ecosystem cost ~2 weeks; integrating a vendor would have cost ~2 weeks of *integration* plus ongoing vendor coordination.

The build cost was bounded. The vendor cost would have compounded. Build won on a clear margin.

## When buy is the right call

To be honest about when *not* to build a Critic:

- **Teams without a release-gating use case.** If eval is exploratory, a vendor's polished UI for browsing runs beats a homegrown gate.
- **Teams whose surface area is wide and shallow.** Vendors invest in dataset management and run tracking; building those from scratch is a year-long project.
- **Teams without a copyright-pressure domain.** If the reference corpus is "internal company data," public-domain provenance doesn't apply and the moat argument weakens.
- **Teams where eval is a side concern, not a product surface.** A Critic-as-a-product is overinvestment if eval is a sidecar.

The Critic is right for *production-gated, customer-facing, audit-bearing* eval in domains with copyright pressure. That's the surface most AI-platform-PM roles in 2026 own — but it isn't every surface.

## The hybrid posture I actually recommend

For most teams reading this case study: **buy an orchestrator, build the policy.**

A pragmatic stack looks like:

- **Orchestration layer** — LangSmith or Braintrust, for run tracking, dataset versioning, eval-run UX.
- **Policy layer** — the equivalent of the Critic, sitting between the orchestrator and the judge providers. Encodes cross-provider non-identity, the rubric grammar, and the reference-corpus provenance.
- **Judge providers** — OpenAI and Anthropic adapters, both registered, with the policy layer routing per call.
- **CI integration** — GitHub Actions calling the policy layer, posting results as PR comments.

The Critic *is* the policy layer. It does not try to be the orchestrator. The architecture deliberately leaves the orchestrator slot empty so a customer can plug in whatever orchestrator they already use.

## What about LangSmith integration specifically?

The Urja Critic's v1 ship does not include a first-party LangSmith adapter. The integration story is *the orchestrator calls the Critic's HTTP API*, which works today with any HTTP-capable runner.

A first-party LangSmith integration is on the roadmap as an adapter (`@urja/critic-langsmith`) — a thin package that registers the Critic as a custom evaluator within a LangSmith project. The shape is straightforward; the priority is bounded by the customer count actually asking for it.

## What the build evidences

For the Senior PM · AI Platforms lane, the build-vs-buy decision evidences:

- **Vendor-landscape literacy.** I know what LangSmith and Braintrust ship and where their gaps are.
- **Discrimination between orchestration and policy.** Most candidates conflate these. Recruiters at applied-AI roles want PMs who can articulate the layer they're building at.
- **Pragmatic build cost discipline.** ~2 weeks for a bounded surface, with a clear contract that absorbs new domains. This is the build-cost profile platform teams actually pay for.
- **Hybrid posture as the recommendation.** Not "build everything, vendors are bad." Build the policy, buy the orchestration. That's the right answer for almost every team — and articulating it concretely separates platform PMs from generalist PMs.

## Decision

**Build the policy layer. Buy the orchestrator if you want one.** The Urja Critic occupies the layer above any orchestrator and below any judge provider. It enforces what orchestrators don't enforce (cross-provider non-identity, public-domain reference corpus, structured rubric grammar) and stays out of the layers other tools handle well (dataset management, run-tracking UX, prompt-comparison playgrounds). The architecture is small enough to build in two weeks, valuable enough that it's the case-study hook, and non-overlapping enough with the vendor landscape that the build-vs-buy decision survives even as the vendor landscape evolves.

---

**Next:** [07 · CI integration](./07-ci-integration.md)

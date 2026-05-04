# 01 · Problem & Users

## The problem in one sentence

Most LLM-as-judge pipelines grade their own family's output, and the literature on self-preference bias has been compounding since 2023 — but the eval tooling shipped today still defaults to single-provider grading.

## Why this matters now (mid-2026)

Four trends converged in the last 18 months:

1. **Generator quality leveled up.** Frontier models hit "good enough on most surfaces" — which makes the *evaluator* the bottleneck, not the generator. If your judge is biased, your release gate is biased.
2. **Eval became a product surface.** Anthropic, OpenAI, Galileo, Braintrust, LangSmith, Arize all shipped eval offerings. Most of them treat the judge model as a *configuration*, not a *policy*.
3. **Copyright pressure on training data hit the eval layer.** The 2024–2025 wave of authorship lawsuits made "what corpus does your evaluator cite?" a procurement question. Judge models that implicitly reference in-copyright textbooks are now a vendor risk.
4. **Generative agents became routine.** When a system prompt, a tool, and a model produce hundreds of artifacts a day, manual review fails. The team needs an automated gate. The gate's bias becomes the system's bias.

The Critic exists because in the system that hosts it (Insights by Omkar — multi-agent consumer SaaS, 1,200+ library entries, 13+ content traditions), the team — *me* — needs every shipped artifact to clear a quality bar that's defensible to a customer's lawyer, not just to my own taste.

## Who needs cross-provider eval

The case study lane is **AI Platform PM and AI safety / eval-track roles**. The Critic is built for myself first, but the design generalizes. Here are the users a recruiter at Anthropic, Sierra, Decagon, Modal, Hebbia, or Mercor would recognize:

| Persona | What they need | What they get from the Critic |
|---|---|---|
| **Eval lead at an applied-AI startup** | An auditable judge that doesn't grade-inflate when the team ships on the same provider it judges with | Structural provider non-identity · score envelope with provider tag |
| **Platform PM at a multi-tenant agent platform** | A gate model that customers can trust across model swaps | Provider-pair calibration suite · the gate doesn't change shape when the generator changes provider |
| **Trust & safety reviewer** | A gate that produces evidence, not vibes | Per-axis reasoning + severity-tagged failings + landmark pointers · audit-trail over time |
| **Vertical AI founder** | Domain-grounded eval where the corpus is legally clean | Public-domain pre-1929 reference corpus with explicit exclusion list |
| **Solo builder with a release pipeline** | A CI-friendly judge that comments on PRs | GitHub Action posts the score table + pass/fail vs threshold |

## What the incumbents get wrong

Three failure modes show up across the eval-tool landscape. The Critic's response to each is structural, not procedural.

### Failure mode 1 · Same-provider judging

Naive LLM-as-judge — the most common pattern — runs the generator and the judge on the same provider, often the same model. The "Judging LLM-as-a-Judge with MT-Bench" paper (2023) and subsequent self-preference work (2024) showed the judge prefers its own family's outputs. Most teams know this and shrug, because cross-provider grading is *more* code, *more* keys, *more* cost.

The Critic refuses to be configurable into that mistake. The default provider is OpenAI today; the engine reads a `CRITIC_DEFAULT_PROVIDER` env, and the route handler refuses to dispatch to a judge whose provider matches the generator's recorded provider. The check is one line in the route handler. There is no "let it slide" flag.

### Failure mode 2 · Configuration as policy

LangSmith and Braintrust expose the judge model as a parameter the developer chooses. Cross-provider grading is achievable — by writing it yourself. The default path is single-provider. Defaults dominate.

The Critic flips the default. Cross-provider is what you get unless you write code to opt out, and there is no opt-out flag in the public API. The customer-facing contract says "the judge will be a different provider than the generator," and the API enforces it.

### Failure mode 3 · Reference corpus by accident

When the judge LLM cites domain knowledge, it cites whatever's in its training set. For most domains that includes textbooks under copyright. For astrology specifically — the vertical the Critic was first built for — that means Hand, Greene, Rudhyar, Hickey, Llewellyn — all post-1929, all under active copyright.

The Critic's reference sets are explicit, versioned data: a manifest of works, landmark targets, and (where licensed) image plates. The corpus floor is **public-domain pre-1929** unless the rights are owned outright. See [05 · Public-domain reference corpus](./05-decision-public-domain-corpus.md) for the line and the exclusion list.

## The non-users

Equally important — who this is *not* for, today.

- **Teams already running mature in-house eval frameworks** with cross-provider grading already encoded. They don't need a policy layer; they have one.
- **Pure benchmarks teams (MMLU, MT-Bench, HELM)** — the Critic is a *production* gate, not a benchmark harness.
- **Pre-revenue side projects** that haven't hit the volume where automated review beats manual review. Cost optimization isn't free; cross-provider grading is ~2× the per-call cost.
- **Domains where the entire reference corpus is post-1929** with no public-domain antecedent. The corpus discipline doesn't generalize cheaply outside humanities + craft domains.

## How the parent SaaS uses the Critic

The Critic was built for the consumer site (`insightsbyomkar.com`). Every illustration, glyph, character render, and scene composition that ships into the site goes through the Critic before it lands in the published library. The site is the Critic's first paying customer — same model the rest of the Urja Visual API follows. See the [parent SaaS case study](https://github.com/Insights-By-Omkar/insights-by-omkar-case-study) for the consumer-side context; this case study covers the eval surface only.

## What the user reads in 30 seconds

Recruiters scrolling LinkedIn Featured tiles see the README. The 30-second test:

1. **TL;DR · paragraph 1** — what the Critic is + the one design decision.
2. **TL;DR · paragraph 2** — why that decision is rare and why it matters now.
3. **At-a-glance table · architecture row** — "cross-provider by design · weighted-rubric · 0–10 anchored scales · severity-tagged failings."
4. **The architecture diagram** in [02 · Architecture](./02-architecture.md) — generator and judge are visibly different boxes, with the provider mismatch enforced at the boundary.

If those four moments don't land the hook, the reader leaves. The case study is structured around the assumption that they will, in fact, leave — which is why the decision documents back the TL;DR up with concrete reasoning.

---

**Next:** [02 · Architecture](./02-architecture.md)

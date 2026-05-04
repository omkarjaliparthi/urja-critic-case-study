<p align="center">
  <a href="https://urja.insightsbyomkar.com"><img src="https://img.shields.io/badge/Live_API-Urja_(Visual_API)-6E56CF?style=for-the-badge" /></a>
  <img src="https://img.shields.io/badge/Built_by-Omkar_Jaliparthi-6E56CF?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Eval-cross--provider_by_design-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-v0.8_·_Spring_2026_launch-brightgreen?style=for-the-badge" />
</p>

<p align="center">
  <i>Public case study for the <b>Urja Critic</b> — a cross-provider taste auditor inside the Urja Visual API. The LLM that grades is, by policy, never the same provider as the LLM that generated.</i>
</p>

---

## TL;DR

The **Urja Critic** is the evaluator inside [Urja](https://urja.insightsbyomkar.com), the Insights by Omkar Visual API. It scores generated artifacts — illustrations, glyphs, character renders, scene compositions — against weighted rubrics anchored to a **public-domain (pre-1929) reference corpus**. Its one defining design decision: **the evaluator and the generator are different LLM providers, by design**. The provider mapping is a one-line policy — `judge.provider !== generator.provider`. Same-family bias becomes an architectural impossibility, not a runtime concern.

This is rare in 2026. Most LLM-as-judge pipelines run a single provider on both sides, inheriting documented self-preference bias. The mainstream eval harnesses (LangSmith, Braintrust, OpenAI Evals, Anthropic Evals) treat cross-provider grading as opt-in glue, not a structural guarantee. The Critic encodes the guarantee into the type system, ships it behind a metered API, and integrates into CI as a pass/fail gate. It is a **policy layer** on top of any harness — not a competitor to one.

---

## At a glance &nbsp; <sub><i>last verified 2026-05-04 · current: v0.8 — Spring 2026 launch</i></sub>

| | |
|---|---|
| **Product** | Cross-provider taste auditor · `POST /api/v1/critic/critique` · live at `urja.insightsbyomkar.com` |
| **Coverage** | 4 rubric tracks (`character-figure`, `character-part`, `illustrate-glyph`, `ambient-effect`) · 2 reference sets at v1.0 (`great-dane`, `canine-animated`) · adding 1 rubric / 1 reference set per release wave |
| **Architecture** | Cross-provider by design · weighted-rubric · 0–10 anchored scales at 2 / 5 / 8 / 10 · severity-tagged failings with `proposedTargetValue` for auto-refine |
| **Reference corpus** | Public-domain pre-1929 line · text + landmark targets + image plates · explicit exclusion list of in-copyright modern authors |
| **Integration** | GitHub Actions (`lucky-critique.yml`) · PR-comment score table · pass/fail gate vs threshold · nightly regression sweep |
| **Stack** | Next.js 16 · App Router · TypeScript strict · scoped JWT auth · Upstash rate-limit · provider adapters behind a single interface |
| **Status** | v0.8 · Spring 2026 launch promo live · beta stability on the API surface · `urja-client` v0.2.0 published to npm |
| **Parent business** | Omkar's Holistic Services LLC (DBA *Insights by Omkar*) · formed May 2023 |

---

## 📑 Read the case study

1. **[Problem & users](./docs/01-problem-and-users.md)** — why cross-provider eval, who needs it, what the incumbents miss
2. **[Architecture](./docs/02-architecture.md)** — generator → output → opposite-provider judge → rubric runner → score envelope → CI
3. **[Decision · cross-provider by design](./docs/03-decision-cross-provider-by-design.md)** — the headline decision · why independence is structural, not optional
4. **[Decision · rubric design](./docs/04-decision-rubric-design.md)** — weighted axes, anchored scales, calibration, parser contract
5. **[Decision · public-domain reference corpus](./docs/05-decision-public-domain-corpus.md)** — pre-1929 line · the copyright trap modern evaluators inherit
6. **[Build vs buy](./docs/06-build-vs-buy.md)** — vs LangSmith, Braintrust, OpenAI Evals, Anthropic Evals, naive LLM-as-judge, human-only review
7. **[CI integration](./docs/07-ci-integration.md)** — PR-comment scoring, pass/fail gates, the developer experience
8. **[Outcomes & lessons](./docs/08-outcomes-and-lessons.md)** — what worked, what didn't, what's next
9. **[Worked example](./docs/worked-example.md)** — end-to-end critique of a generated illustration · per-axis scores, evidence, cross-provider tag

---

## Related artifacts

- **[Live API · Urja](https://urja.insightsbyomkar.com)** · [Pricing](https://urja.insightsbyomkar.com/pricing) · [Critic track docs](https://urja.insightsbyomkar.com/docs)
- **[urja-client · npm](https://www.npmjs.com/package/urja-client)** — type-safe URL builders + React ThemeProvider for Urja (v0.2.0)
- **[Kriya case study](https://github.com/omkarjaliparthi/insights-astrology-api-case-study)** — sibling commercial astronomy API · 109+ endpoints from first principles
- **[Parent SaaS case study](https://github.com/omkarjaliparthi/insights-by-omkar-case-study)** — the consumer product that dogfoods both Urja and Kriya
- **[Insights by Omkar](https://www.insightsbyomkar.com)** — the consumer SaaS · Urja's first paying customer

---

## Skills this project evidences

<table>
<tr>
<th>Product</th>
<th>Program</th>
<th>Engineering</th>
<th>Business</th>
</tr>
<tr>
<td valign="top">

- AI eval product design
- Rubric-as-product surface
- Anchored-scale calibration
- CI as developer experience
- Policy layer vs harness positioning

</td>
<td valign="top">

- Cross-provider eval governance
- Reference-set versioning
- Stability tiers (experimental → beta → stable)
- Release-gating discipline (n=2)
- Provenance-clean training data

</td>
<td valign="top">

- Provider-adapter interface design
- Cache key on full (rubric × ref × provider × output) tuple
- Type-system enforcement of provider non-identity
- Structured-output parser with severity tags
- Stateless scoped JWT + rate-limit edges

</td>
<td valign="top">

- Build vs buy on eval orchestration
- Copyright-clean corpus as moat
- Cost discipline at eval time (sample vs gate)
- Pricing on metered `:use` scope
- Defensibility in front of customer legal

</td>
</tr>
</table>

---

## What's not in this repo

This is a write-up. Not a release.

- **No proprietary code.** No source from `lib/critic/`. No route handlers. No prompt templates.
- **No real rubric weights.** The illustrative weights in this case study are representative, not the ones running in production.
- **No engine internals.** The cache-key composition, the parser's failure-recovery rules, and the provider-adapter implementations stay private.
- **No customer list.** Use cases discussed here are hypothetical, not commitments.
- **No specific provider versions used internally** beyond what's already on the public pricing page.

The architecture, the policy, the rubric grammar, and the build-vs-buy reasoning are public. The implementation is not. That separation is deliberate — what's defensible about the Critic is the *policy*, and the policy is what this case study is for.

---

<p align="center">
<b>Hiring Senior PM · AI Platforms · Founding Platform PM (Series B+)?</b><br/>
<a href="mailto:admin@insightsbyomkar.com">admin@insightsbyomkar.com</a> · <a href="https://www.linkedin.com/in/jaliparthiomkar">LinkedIn</a> · <a href="https://github.com/omkarjaliparthi">GitHub</a>
</p>

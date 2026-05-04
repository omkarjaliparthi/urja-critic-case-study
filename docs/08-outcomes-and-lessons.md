# 08 · Outcomes & Lessons

## What shipped at v0.8

- `POST /api/v1/critic/critique` — billable, scoped, rate-limited, cross-provider gate
- `GET /api/v1/critic/rubrics`, `/rubrics/:id`, `/references`, `/references/:id` — inspection surface, free tier
- 4 rubric tracks live (`character-figure`, `character-part`, `illustrate-glyph`, `ambient-effect`)
- 2 reference sets at v1.0 (`great-dane` v1.1.0, `canine-animated` v1.0.0)
- Provider adapter pattern with OpenAI registered today, Anthropic adapter behind the same interface
- `lucky-critique.yml` GitHub Actions workflow — PR comments, gate, nightly regression
- `urja-client` v0.2.0 published to npm (URL builders + React ThemeProvider; not the Critic adapter — that's v0.3 roadmap)
- Public-domain reference corpus policy with explicit exclusion list, audit-trail published
- Anchored 0–10 scoring with calibration anchors at 2 / 5 / 8 / 10
- Severity-tagged failings (1–5) with `proposedTargetValue` for auto-refine
- Stability tiers (`experimental` / `beta` / `stable`) on rubrics, surfaced in the listing endpoint
- Spring 2026 launch promo — 30% off first 3 cycles, ends 2026-07-31

## Where it is now &nbsp; <sub><i>(as of 2026-05-04 · v0.8 · launch promo live)</i></sub>

- **Cross-provider policy in production.** Every PR-merge gate runs cross-provider. Provider attribution lands in every PR comment.
- **Auto-refine loop closed on numeric failings.** The character-figure rubric's `autoApplicable: true` failings drive a refinement loop that has shipped multiple character iterations without human intervention beyond final review.
- **Reference-set versioning honored.** Bumping `great-dane` from 1.0.0 to 1.1.0 (added Lucky photographs as image references) invalidated the right cached scores; scores shifted in the documented direction (more specific failings, more grounded narratives).
- **The rubric grammar absorbed two new tracks** (`illustrate-glyph`, `ambient-effect`) without engine changes. The rubric runner is rubric-agnostic; new domains land as data, not code.

The point of this section: v1 is the gate, not the dashboard. The dashboard is where most teams start; the Critic deliberately starts at the gate because a gate that earns developer trust justifies the dashboard, while a dashboard that earns no developer trust justifies nothing.

## What this proves

Not "one person can build an eval framework." That would be misleading.

The actual lesson: **policy is more durable than implementation.** Every line of code in `lib/critic/` will be rewritten over the next 24 months as provider adapters change shape, judge models update, and the rubric grammar expands. The cross-provider policy and the public-domain corpus policy will not be rewritten — they're the design's load-bearing wall. The case study is structured around the policy, not the code, because the policy is what survives.

If a recruiter at an AI-platform team is reading this, the question worth asking is *"would this policy still be the right answer if every line of code were rewritten?"* Yes. That's the test of whether a decision is structural or incidental.

## What the rubric grammar learned

The first rubrics shipped with continuous 0–10 scales and a freeform "explain your reasoning" field. Both produced unreliable scores.

The fix:

1. **Anchored scales at four points** (2 / 5 / 8 / 10). Run-to-run consistency tightened immediately. Median axis variance across two runs dropped roughly 3× by eye — formal numbers will land in a v1 anniversary post.
2. **`anchorBand` field in the response** (`"5→8"`). Forced the judge to *commit* to where on the scale the output sat, not just emit a number.
3. **Severity-tagged failings instead of paragraph narratives.** Paragraph narratives were beautiful and useless. Severity tags are ugly and actionable.
4. **`proposedTargetValue` as its own field, not a regex over `proposedFix`.** Added in a v1.1 rubric bump after the auto-refine loop tried to extract `0.50` from `~0.50` and fed `0` to the generator.

The grammar matured by deletion as much as by addition. Most of the v0.x → v0.8 work was *removing* the bad fields the v0 rubrics had.

## What the cross-provider policy learned

- **Calibration suite is non-negotiable.** Without provider-pair calibration, swapping judges produced visibly different scores for the same input. The fix was a fixed set of historical artifacts re-scored under each registered provider before that provider becomes eligible for that rubric.
- **Fail-open is the right call on judge outages.** The original implementation failed-closed (treated provider 5xx as a critique failure). At a 2 AM judge outage during a deploy push, that was operationally bad. Switching to fail-open-into-manual-review eliminated the false-deploy-block class of incident.
- **Provider attribution is more important than the score itself.** When customers ask about a score, the question is almost always *"who graded this?"* The provider tag in every envelope and every PR comment short-circuits 80% of the conversations.

## What the public-domain corpus learned

- **The exclusion list is the load-bearing artifact.** The list of *included* sources is interesting; the list of *excluded* sources is auditable. Customer legal teams care about the exclusions far more than the inclusions.
- **`fair-use-critique` has to be narrow.** The first iteration of the `canine-animated` reference set used images more permissively. Tightening to a small, documented set with a fair-use rationale per item was the right discipline. If the rationale wouldn't survive a customer's legal review, the reference doesn't ship.
- **Reference-set versioning surfaces score shifts.** When a reference set bumps, scores can shift in the same direction the bump intended. Surfacing the version in every envelope means clients know when a shift is expected vs when it's regression.

## What worked

**The provider adapter interface.** Two adapters today, both behind the same interface. Adding a third (Anthropic native, today behind the unified interface) is a sibling file, not a refactor. This is the same shape that made [Kriya's three SDKs](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study/blob/main/docs/06-sdk-strategy.md) tractable — small surface, single interface, swap-don't-extend.

**The cache key on the full tuple.** Including `rubric_version`, `reference_set_version`, and `provider_id` in the cache key meant cache invalidation was automatic on every bump. No "did we forget to invalidate the cache?" incidents. The cache is honest about what determines the response.

**Reusing the auth + rate-limit patterns from sister APIs.** The scoped JWT pattern and Upstash-with-fallback rate-limit pattern were ports from Kriya, not inventions. Cross-project pattern reuse is the single most productive habit a solo builder has — same lesson Kriya's case study calls out.

**The `severity` field on failings.** Surfaces the gate-blocking issues immediately. Severity 5 fails the gate regardless of weighted total; severity 1–2 are nits that don't block. Triage at parse time, not by humans reading paragraphs.

**Treating the PR comment as the product.** The score envelope is data; the comment is the product. Investing in comment formatting (provider attribution at the top, anchor bands per axis, failings ordered by severity) was the single highest-ROI v0.x decision.

## What broke

**v0 rubrics shipped without anchors.** Run-to-run drift was bad enough to block production gating until anchors went in. Two-week detour that should have been the v0 ship state.

**Cost telemetry was thin until v0.5.** The `usage.costUsd` field arrived late. Before it, "is this gate expensive?" was a vibe, not a number. Should have been the day-one envelope field.

**Customer-facing rubric override path.** v1 doesn't ship one. Customers either use a public rubric or commission a custom one (private engagement). At enterprise volume this won't survive — overrides will need to ship as a versioned customization layer. Roadmap.

**Anthropic provider adapter was scoped for v1.** Shipped behind the unified provider interface but not registered as a default judge. Activating it for production traffic is a v0.9 task, gated by the calibration suite.

**No first-party LangSmith / Braintrust adapter.** The HTTP API works with any orchestrator, but the integration story would land harder with a first-party adapter package. Roadmap, gated by customer demand.

**Documentation gap on the auto-refine loop.** The loop works against `autoApplicable: true` failings, but the public docs don't yet walk through the developer-side integration of "score → apply target → regenerate." That's a worked-example post that hasn't been written.

## What I'd do differently

- **Ship anchored scales on day one.** The v0 detour through continuous scales taught nothing the literature didn't already say.
- **Surface provider attribution in v0, not v0.5.** Customers want to see the provider tag from the first envelope they read. Withholding it cost trust at the first few customer conversations.
- **Build the calibration suite before the second provider.** I added the second provider adapter first and then realized I needed the calibration suite to certify it. Reverse order would have shipped the calibration discipline earlier.
- **Write the source policy before the first rubric.** The exclusion list is more durable than any individual rubric. Writing it first would have set the corpus discipline from line one.

## What's next — priorities post-v0.8

1. **Anthropic adapter promoted to a default judge** — gated by calibration suite drift bounds
2. **Worked-example post for the auto-refine loop** — public-facing developer doc
3. **First-party LangSmith adapter** (`@urja/critic-langsmith`) — thin package, registers Critic as an evaluator
4. **Per-path threshold support** — higher bar for hero images, lower for utility glyphs
5. **Slack-notification path** for failed gates — alongside the existing PR comment
6. **Public dashboard for score history** — once the gate has earned developer trust on the PR-comment surface
7. **Trajectory-input rubrics** — extending the rubric grammar from image inputs to agent-trajectory inputs
8. **Public calibration-suite doc** — describe the drift-tolerance rules without leaking the suite's contents

## Metrics to check at the 30-day post-launch mark

The Spring 2026 launch promo runs through 2026-07-31. The 30-day check-in (early June) targets:

- **Critique calls per week (across all customers)** — target 5,000
- **Cache hit rate on the gate path** — target ≥ 30% (high cache hit means PRs are iterating against the same artifacts; healthy)
- **Median elapsed time per critique** — target < 6 seconds (cold provider call dominates)
- **PR-gate pass rate** — target 60–75% (too high = gate is too lenient; too low = rubric is mis-tuned)
- **Manual-review-queue size** — target < 20 items / week (judge-outage fail-opens + structural failings combined)
- **Customer count on `:use` scope** — target 25
- **Cross-provider attribution mismatches** — target 0 (any same-provider grading is a bug)

If the cross-provider attribution mismatch metric is anything other than zero, the case study is wrong about the architecture and gets rewritten. That's the fundamental health check.

## Portfolio value

This case study, paired with the [Kriya astronomy API](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study) and the [parent SaaS](https://github.com/Insights-By-Omkar/insights-by-omkar-case-study), covers three layers of the AI-platform-PM surface:

- **Insights by Omkar (consumer SaaS)** — evidence of shipping a full AI consumer product with paying users, multi-agent governance, dual payment rails, real chargeback defense.
- **Kriya (commercial astronomy API)** — evidence of deep technical work (astronomy math from primary sources), dev-tool product thinking (three SDKs, zero deps, MIT licensing), and commercial API design.
- **Urja Critic (this case study)** — evidence of AI-platform thinking specifically: cross-provider eval policy, public-domain reference corpus, structured rubric grammar, CI integration as developer experience.

Few Senior PM / Founding PM candidates have all three. Fewer have written them up at this depth. That's the portfolio bet.

## What I want recruiters to take from this

If you're hiring a Senior PM for AI Platforms, or a Founding Platform PM at Series B+, the questions worth probing me on after reading this:

- *"Walk me through the calibration-suite design for adding a third provider."*
- *"How does the rubric grammar generalize to non-visual eval surfaces?"*
- *"What's the migration path for a team running same-provider eval today?"*
- *"How do you price metered eval against expected customer cost-tolerance?"*
- *"What's the v2 dashboard look like, and why is it v2 instead of v1?"*

I have specific answers to each. The case study is the proof that the questions are worth asking.

---

**Back to:** [case study index](../README.md)

**Also:** [Kriya case study](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study) · [Parent SaaS case study](https://github.com/Insights-By-Omkar/insights-by-omkar-case-study) · [Live API · Urja](https://urja.insightsbyomkar.com) · [urja-client v0.2.0](https://www.npmjs.com/package/urja-client)

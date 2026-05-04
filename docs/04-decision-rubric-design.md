# 04 · Decision · Rubric Design

## The question

How do you turn "is this output good?" into a number a CI gate can trust?

The naive LLM-as-judge approach asks the model to "rate this 0–10." That produces high-variance scores, inconsistent across runs, and untrustworthy as a release gate. The Critic's rubric design is the answer to *what specifically* is wrong with naive grading and how to fix each part.

## What naive grading gets wrong

Three failure modes show up in single-axis 0–10 grading:

1. **Score collapse.** Without anchors, models cluster scores between 6 and 8 regardless of input. A 7 means nothing because 7s arrive for both excellent and mediocre work.
2. **Run-to-run drift.** The same input scored twice, an hour apart, returns different numbers because the prompt did not constrain the scoring boundary.
3. **No actionability.** A score of 6 tells the developer *something is wrong* but not *what* and not *what to change*. The refinement loop has no fuel.

The rubric system fixes each.

## The grammar — five fields per rubric

Every rubric is a structured data record (no free-form prompts):

| Field | Purpose | Why it matters |
|---|---|---|
| `axes[]` | Ordered list of evaluation dimensions | Splits one-number grading into multiple specific numbers |
| `weight` per axis (0..1, sum to 1) | What matters most to the gate | Forces explicit prioritization |
| `anchors` per axis (at scores 2 / 5 / 8 / 10) | Concrete descriptions at four points on the 0–10 scale | Calibrates the model · the floor-post trick |
| `guidance` per axis | What to look for when scoring this axis | Tells the model what it's actually evaluating |
| `globalGuidance` | Cross-axis rules · tie-breaks · named failure modes | Catches "looks pretty but means the wrong thing" |

The judge LLM is told: *"Use the anchor descriptions as floor posts. Describe why the output falls between or on an anchor. Scores 0, 1, 3, 4, 6, 7, 9 are reserved for between-anchor scoring."* That single instruction collapses run-to-run drift more than any other mitigation tested.

## Worked example — illustrate-glyph rubric

Illustrative shape (real weights are private):

```
Rubric: illustrate-glyph
  axes:
    1. silhouette-read-at-24px (high-weight)
    2. line-weight-hierarchy (mid-weight)
    3. construction-clarity (mid-weight)
    4. craft-fidelity (mid-weight)
    5. semantic-honesty (high-weight)
```

Each axis has anchors at 2 / 5 / 8 / 10. For `silhouette-read-at-24px`:

| Score | Anchor description |
|---|---|
| **2** | Glyph degrades into "two slashes" or "a blob" at 24×24 — only readable above 48×48 |
| **5** | Recognizable at 32×32 but not at 24×24 |
| **8** | Recognizable at 24×24 with mild squinting |
| **10** | Instantly recognizable at 24×24 even with bad subpixel hinting |

The judge places the output between anchors, not at an arbitrary point on a continuous scale. A 7 means *between 5 and 8, closer to 8*. A 6 means *between 5 and 8, closer to 5*. The anchor band is recorded in the envelope (`"anchorBand": "5→8"`) for audit.

## Anchored scales beat continuous scales

The single most counterintuitive finding from running the Critic against real artifacts: **fewer numbers, more reliably placed, beats more numbers placed sloppily**. The 0–10 scale with four hard anchors produces tighter run-to-run consistency than a 0–100 scale or an open "rate this from 1 to 5" scale.

Why: the model is not a continuous function. It treats labels (the anchor descriptions) as categories and interpolates between them. The interpolation is consistent because the categories are stable. A continuous scale forces the model to invent a calibration on the fly, and the invented calibration drifts.

This is the same lesson human-rater literature has — Likert scales with anchored endpoints (1 = "strongly disagree", 5 = "strongly agree") outperform continuous sliders for cross-rater consistency.

## Weighted total · server-side, never client-side

The weighted total is computed server-side from the per-axis scores. **Clients do not compute it locally.** Two reasons:

1. **Calibration drift containment.** If the rubric weights tune in a minor SemVer bump, every cached envelope's weighted total recomputes to the new weights. Clients computing locally would silently ship stale totals.
2. **Dispute simplification.** When a customer asks *"why did this score 7.2?"* — there's one answer, recoverable from the envelope. No "well, depends on which client version computed it."

The weighted total is rounded to one decimal place. The per-axis scores are 0.5-step integers. That granularity is the highest precision the calibration suite can reliably distinguish — finer granularity is noise, not signal.

## Failings — the actionability layer

A score is not enough. The rubric runner also produces a list of **failings** — discrete items the generator can act on:

| Field | Purpose |
|---|---|
| `title` | Short imperative — "muzzle too short" / "missing tuck-up" |
| `axisId` | Which rubric axis this rolls up to |
| `landmarkId?` | Which reference landmark this violates (if applicable) |
| `severity` | 1 (nit) → 5 (breed-breaking / ship-blocking) |
| `proposedFix` | Concrete imperative — "increase muzzleLengthRatio from ~0.30 to ~0.50" |
| `autoApplicable` | Boolean — is the fix a numeric parameter adjustment, or structural? |
| `proposedTargetValue?` | The number the parameter should be — required when `autoApplicable: true` |

**Why `proposedTargetValue` is its own field, not a regex over `proposedFix`:** the auto-refine loop needs a structured value to update the manifest with. Regex-extracting numbers from prose is a parser-error generator. This was a v1.1 change to the rubric type — added after the auto-refine loop tried to extract `0.50` from `"~0.50"` and fed `0` to the generator.

The actionability layer is what turns the Critic from a pass/fail oracle into a refinement-loop driver. The generator iterates until the failings drop below severity 3 across the rubric.

## Stability tiers — experimental → beta → stable

Rubrics ship with a stability tag:

- **`experimental`** — newly authored. Allowed to be referenced from the API but not yet recommended for gates. Score consistency across runs has not been validated on n ≥ 3 distinct subjects.
- **`beta`** — passed n ≥ 3 stable results across distinct subjects. Recommended for gates with a documented threshold.
- **`stable`** — production-validated across the domain. Used in customer-facing gates without caveats.

A rubric tuning (weight changes) is a minor SemVer bump and stays in the same stability tier. Adding axes is minor. **Removing or renaming axes is a major bump** and resets the stability ladder — the rubric is `experimental` again until re-validated.

This discipline is the same shape as the SDK release-gating pattern in [Kriya's astronomy API](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study/blob/main/docs/06-sdk-strategy.md) — public surface promotes only after the underlying validation is in place.

## The parser contract

The judge LLM returns structured JSON. The parser validates the response against the rubric's output schema and **clamps**, not rejects, on the most common drift modes:

| Drift mode | Parser behavior |
|---|---|
| Score outside 0–10 | Clamp · log warning |
| Score not on 0.5 step | Round to nearest 0.5 · log warning |
| `severity` outside 1–5 | Clamp · log warning |
| Missing `axisId` for an axis the rubric defines | **Reject** — re-prompt or 5xx |
| Failings list empty when weighted total < 9.5 | **Reject** — re-prompt with reminder |
| `autoApplicable: true` without `proposedTargetValue` | **Reject** — schema violation |
| Extra unknown fields in the response | Ignore · log info |

**Clamp where the model is approximately right; reject where the model has dropped a contractual field.** This is the rule that keeps the parser working as judge models change generation. Strict parsing on every drift would mean every model upgrade breaks the gate. Loose parsing on missing axes would mean silently incomplete gates.

## What's not in the rubric grammar

- **No free-form prose criteria.** Every criterion is an axis with anchors. Prose-only criteria are not addressable in the parser.
- **No multi-modal weights** (e.g. "weigh axis X higher when axis Y > 8"). Conditional logic across axes is in `globalGuidance`, not the weight schema. The flat weighted sum is intentionally simple.
- **No per-customer rubric overrides shipped at v1.** Customers either use a public rubric or commission a custom one (private). Allowing per-call rubric tweaks would invalidate every cache and break score reproducibility.
- **No agent-trajectory rubrics yet.** The rubric grammar generalizes; the v1 ship is image-input rubrics. Trajectory-input rubrics are on the v2 roadmap.

## What I'd change if starting over

- **Anchor at 5 points instead of 4** — adding an anchor at 6 helps the model distinguish "barely above the gate" from "comfortably above the gate." The four-anchor scheme is a v1 simplification.
- **Make `globalGuidance` a structured field**, not free-form prose. Right now it's freeform text the prompt builder concatenates; structured rules (e.g. "if axis X < 5, override total to fail") would be parser-checkable.
- **Ship a rubric linter** that validates the rubric structure before registration — weights summing to 1.0 within tolerance, all anchors present, axisIds matching the schema. Today this happens at module load and throws; a CLI linter would catch errors at PR-time.

## Decision

**Rubrics are versioned data, not prompts.** Five-axis structure, weights summing to 1.0, anchors at 2 / 5 / 8 / 10, severity-tagged failings with `proposedTargetValue` for auto-refine, parser that clamps on drift but rejects on contract violation. Stability tiers gate which rubrics are used as production gates. The grammar is what makes the cross-provider policy meaningful — without rubric structure, "cross-provider grading" is just two random numbers.

---

**Next:** [05 · Decision · public-domain reference corpus](./05-decision-public-domain-corpus.md)

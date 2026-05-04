# Cover image spec

Drop a `cover.png` in this directory at the spec below. The README does not
embed it at v1 (intentionally — Kriya case study leads with the badge row,
not an image). Use the cover for the LinkedIn Featured tile thumbnail.

## Dimensions

- **1200 × 630 px** (LinkedIn / OpenGraph standard)
- PNG, sRGB, 144 dpi

## Type

- **Headline** (top-third, large): `Urja Critic — Cross-Provider Taste Auditor`
- **Subtitle** (mid-third, smaller): `Evaluator and generator are different LLM providers, by design`
- **Footer** (bottom-third, smallest): `Insights by Omkar · Visual API · v0.8`

## Visual treatment

- Match the Kriya case study's existing palette (deep indigo / brass /
  parchment).
- Two boxes side by side near the headline — one labeled "Generator
  (Claude)" and one labeled "Judge (GPT)" with a `≠` symbol between
  them. That single visual carries the cross-provider hook at a glance.
- No emojis on the cover. The case study uses section-anchor emojis
  inside the markdown, but the cover stays type-only.

## Why no embedded cover at v1

The README leads with the badge row + TL;DR, matching the Kriya case study.
A cover image embedded above the badges would push the TL;DR below the
fold on most screen sizes. The cover earns its keep on LinkedIn Featured
and as the OpenGraph image; it does not need to live in the README.

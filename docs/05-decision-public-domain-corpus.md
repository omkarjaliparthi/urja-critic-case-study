# 05 · Decision · Public-Domain Reference Corpus

## The question

When the judge LLM cites domain knowledge during grading, where does that knowledge come from?

The default answer most teams ship: *whatever's in the model's training set, implicitly.* Which means the judge cites whatever modern textbooks the foundation model was trained on — most of which are under active copyright.

The Critic's answer: **explicit, versioned reference sets, restricted to the public-domain pre-1929 line** unless the rights are owned outright.

This is the case-study decision recruiters at AI-safety teams pattern-match the hardest. "Training-data provenance for eval corpora" is a 2026 hot topic; the Critic answers it concretely.

## Why this matters

Three pressures landed in 2024–2025 and will only intensify:

1. **Authorship lawsuits.** The wave of class-action suits against foundation-model providers made every downstream user of those models indirectly exposed to the training-data question. Judge models trained on in-copyright corpora carry the exposure into the eval layer.
2. **Procurement scrutiny.** Enterprise procurement for AI tooling now routinely asks *"what data does your evaluator reference?"* — a question that didn't exist on RFPs in 2022.
3. **Customer-legal review.** When a customer's general counsel reviews an eval pipeline, they want to know what authoritative sources the gate cites. "The judge LLM knows things" is not an acceptable answer.

The Critic exists for surfaces (illustrations, glyphs, character renders for an astrology-domain consumer SaaS) where the modern reference corpus is heavily encumbered. The decision generalizes — every humanities-adjacent vertical has the same shape.

## The corpus floor — public-domain pre-1929

US copyright rule applied: works are treated as public domain if **published before 1929** OR if the author has been **deceased ≥ 95 years**. Translations are evaluated independently of the original — a 2nd-century Greek source may be PD while a 2010 English translation of it is not.

For the astrology-vertical content the Critic was first built for, this means the corpus draws on:

- **Claudius Ptolemy**, *Tetrabiblos* (2nd c. CE) — Greek original; English via J. M. Ashmand (1822)
- **Vettius Valens**, *Anthology* (2nd c. CE) — Greek original; English via Mark Riley (released to PD by translator)
- **Manilius**, *Astronomica* (1st c. CE) — Latin original
- **Firmicus Maternus**, *Mathesis* (4th c. CE) — Latin original
- **Guido Bonatti**, *Liber Astronomiae* (13th c.) — Latin original
- **William Lilly**, *Christian Astrology* (1647)
- **Jean-Baptiste Morin**, *Astrologia Gallica* (17th c.)
- **Alan Leo** (d. 1917), collected works
- **Sepharial / Walter Gorn Old** (d. 1929), collected works
- **Charles E. O. Carter**, pre-1929 publications

For the visual / craft rubrics:

- 19th-century engraved astronomy texts and celestial atlas plates
- Late-1800s observatory illustrations
- Public-domain breed-standard plates and anatomical references
- William Morris pages and Art Nouveau ornament references (c. 1900–1920)

## What's explicitly excluded

The exclusion list is published. It is part of the case study's audit trail. For the astrology vertical:

- **Robert Hand**, all works (PD ~2080+)
- **Liz Greene**, all works
- **Stephen Forrest, Steven Arroyo, Demetra George, Bernadette Brady, Sue Tompkins**
- **Robert Schmidt** — including all Project Hindsight translations (PD ~2088)
- **Dane Rudhyar**, all works (PD ~2055)
- **Marc Edmund Jones** — including the Sabian symbols (PD ~2050)
- **Reinhold Ebertin** — including the cosmobiology midpoint catalog (PD ~2058)
- All modern English translations of ancient sources published after ~1929

When a doctrine widely associated with these authors is referenced (e.g., midpoint analysis, time-lord systems, symbolic-degree work), the rubric describes the doctrine using public-domain antecedents and in our own words, never reproducing the modern formulation's language.

## Two non-PD provenance categories — and their rules

Not every reference is public-domain. Two narrow exceptions are allowed:

### `commissioned-owned`

Personal photographs, custom illustrations, and other works where the rights are owned outright. Example: photographs of the real Great Dane the character renderer aspires to (Lucky), used in the `great-dane` reference set.

The rule: full rights retained, license documented in the reference-set manifest, audit-trail to ownership.

### `fair-use-critique`

The narrowest exception. Image references used under 17 USC §107 transformative critical analysis — the *images themselves* are not redistributed; they're consulted by the judge LLM as targets to compare against. Example: a small set of cartoon-craft references used by the `canine-animated` reference set, where the rubric's purpose is *critical evaluation of generated craft against an authoritative reference register*.

The rule: no commercial redistribution of the references themselves, fair-use rationale documented per item, conservative scope. If a reference set's fair-use rationale would not survive a customer's legal review, it doesn't ship.

These two categories are *narrow*. The default floor is public-domain.

## Why this is a moat, not just a discipline

Most competitors who ship eval gates over creative work scrape modern textbooks for the judge's grounding — explicitly or by inheritance from the foundation model. Three consequences follow:

1. **They can be sued.** A class-action against a competitor for ingesting in-copyright material into the eval surface is a foreseeable 2026–2027 event.
2. **Their gate's grounding cannot be audited.** "The judge LLM knows things" is not an audit answer. Customer legal teams cannot get comfortable with implicit grounding.
3. **They cannot license the gate to enterprise customers** who require provenance disclosure as a procurement gate.

The Critic's reference corpus is **explicit, versioned, and copyright-clean** by construction. That is a defensibility property — and defensibility properties are what enterprise procurement pays a premium for.

## The audit trail — `SOURCES.md` published, externally

The Insights Astrology API (Kriya) ships a [SOURCES.md](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study/blob/main/SOURCES.md) listing every public-domain work the rubric corpus draws on, every modern author explicitly excluded, and the legal rationale. The Urja Critic shares the same source policy and audit document.

The audit trail is a **product feature**, not paperwork. A customer's legal team can check in one place which texts are drawn on, which are deliberately not, and why. If a reviewer or a concerned author believes a rubric reproduces in-copyright material, the contact path is documented.

This is the same logic that drives the PR comment style in [07 · CI integration](./07-ci-integration.md): make the gate's reasoning *visible* by default.

## Why "philosophically older, less mode-collapsed"

A side benefit of the pre-1929 floor: the public-domain corpus is older, sparser, and more idiosyncratic than the modern textbook stack. Modern astrology textbooks have converged on a relatively narrow consensus shaped by the 20th-century English-language industry (Llewellyn, Aquarian, etc.). Pre-1929 sources are heterogeneous — Manilius does not agree with Lilly on house meaning, Ptolemy contradicts Bonatti on aspect rules, and the post-1929 consensus is a flattening, not a synthesis.

Grounding the judge in the older heterogeneous corpus produces less mode-collapsed grading. The judge cites multiple traditions and acknowledges disagreements, where a modern-textbook-grounded judge produces a single confident answer. For a domain where *interpretive plurality is part of the doctrine*, the older corpus is more accurate, not just safer.

This is a domain-specific argument. It generalizes to any humanities domain where the post-1929 corpus has flattened the older intellectual tradition — which is most of them.

## Trade-off — what the policy costs

- **Coverage gaps.** Some doctrines that are well-developed in modern literature (midpoint analysis, harmonic charts, symbolic-degree work) have weak pre-1929 antecedents. The rubric corpus describes the doctrine in our own words from public-domain antecedents — which is more work than citing the modern source.
- **Image plate scarcity.** Public-domain image plates are scarcer and lower-resolution than modern reference photography. The `commissioned-owned` and narrow `fair-use-critique` categories exist to fill gaps where the public-domain image base is insufficient.
- **No fast-following on modern fashion.** When a new astrology methodology becomes popular through a 2020s textbook, the Critic does not adopt it until either the doctrine is described from public-domain antecedents in our own words, or the methodology has been put in the public domain by its author.

The trade-off is acceptable because the alternative is being uninsurable as a vendor.

## How this generalizes outside astrology

The decision pattern transfers cleanly:

| Domain | Public-domain floor | Modern excluded |
|---|---|---|
| **Music theory eval** | Pre-1929 treatises (Tovey, Schenker early works), 19th-c. harmony texts | Schenker post-1929 estate works · Schoenberg's *Theory of Harmony* English trans. |
| **Visual craft eval** | Beardsley, Mucha, Morris, Klimt's plates, observatory engravings | Disney character bibles · Pixar art-of books · modern brand guides |
| **Code-style eval** | Knuth pre-1995 vol. I (mostly) · public RFCs | Modern style guides under publisher copyright |
| **Prose-style eval** | Strunk *Elements of Style* (1918) · pre-1929 manuals | Modern editorial guides · publisher house styles |

Every applied-AI vertical that has a copyright-encumbered modern corpus has this decision in front of it. Most are pretending the question doesn't exist. The Critic's answer is to publish the policy and the exclusion list.

## Decision

**Public-domain pre-1929 reference corpus, with narrow exceptions for `commissioned-owned` and `fair-use-critique` categories, all documented in a published source policy.** The corpus is data, versioned, and the bumps are visible to clients via the reference-set version field on every score envelope. The decision is moat, audit, and defensibility in a single policy.

---

**Next:** [06 · Build vs buy](./06-build-vs-buy.md)

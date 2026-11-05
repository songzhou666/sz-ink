English | **[简体中文](README.md)**

# sz-ink (研墨) — Chinese Text De-AI-ifier

> Diagnose the whole piece, cut only at the exact points, protect every anchor, stop the moment it passes.

**Why "grinding ink"**: text produced by an AI is a lump of dry ink — it has to be ground before it can be written with a human hand. The name is a pun: this skill's discriminating power comes entirely from **Chinese language and literature research** (stylistics, lexicology, rhetoric, discourse studies), not from a banned-word list.

sz-ink is a **post-editing rewriter** for Chinese text. It takes writing with an obvious "AI accent" and returns the rewritten text plus a metrics report, a change log and a drift report. It does not generate new content, it makes no promise about defeating AI detectors, and its acceptance criterion is simply "reads like a person wrote it".

---

## How it differs from what is already out there

Most de-AI-flavor tools follow one of three familiar paths: swap synonyms to raise perplexity, hand the model a banned-word list with instructions not to use those words, or aim at fooling a detector. Their shared weakness is that they **only subtract and never restructure**, have no linguistic grounding, and stop working as soon as the model or the word list changes.

sz-ink's discriminating power comes from established findings in Chinese linguistics, each converted into a decidable rule:

| Knowledge source | Becomes this capability |
| --- | --- |
| **Stylistics**: register is a product of context; form that mismatches occasion is improper | **Register profile + conditional thresholds.** There is no global threshold: removing AI flavor from an official document is not the same as making it colloquial; an explanatory register grants terminology exemptions and permits, without requiring, sentence-final particles |
| **Lexicology**: written/spoken opposition; synonyms differ in register colouring | **84 replacement pairs annotated by register** — a word swap must also swap register, otherwise mechanical synonym replacement produces ungrammatical output |
| **Rhetoric**: sentence-pattern choice (balanced vs. loose); conditions for using figures | Rules for **runaway parallelism, same-shell list items and antithetical closings** — the AI's "too neat" versus a human's "loose" is a rule difference, not an aesthetic preference |
| **Grammar**: sentence types (declarative/interrogative/exclamatory); modal meaning of particles | **Tone warmth and whitespace metrics** — 啊/吧/呢/嘛/呗 each carry explicit modal meaning, so the human fingerprint has semantic grounding |
| **Discourse and information structure** | **"Harmless-if-deleted" test for empty paragraphs, structural regularity, perspective-layer lock** — discourse-level defects cannot be cured by deleting words, because the disease is in information organisation |

This lets it distinguish three structurally different kinds of AI flavor: **jargon type** (empower / closed loop / in conclusion), **copywriter type** (poetic shell + runaway parallelism + zero factual anchors) and **rewrite-laundering type** (details identical to public material, enumeration order preserved). Word-list approaches are blind to the latter two.

---

## How it works (S0–S8 state machine)

```
S0 register detection -> S1 profiling -> S2 diagnosis -> S3 pass/fail check
                        below threshold |            | passed
        S4 targeted intervention (<=3 per round) -> S5 retest -> S6 restoration audit -> S7 rollback (on drift)
                                                          |
                                                        S8 deliver
```

- **Diagnosis**: nine statistical metrics (sentence-length rhythm, cliché density, tone warmth, whitespace, sentence-type entropy, structural regularity and more), implemented with the Python standard library, zero dependencies; `--self-test` ships with regression checks.
- **Judgement**: red-line breaches must be fixed; a total of 9 is enough to pass — **all-green is a signal of over-editing**. There is a "over-optimisation" ceiling, because a person who tries too hard to sound human is just acting.
- **Intervention**: each round changes at most the three worst deviations, following prescription cards (symptom → criterion → action → worked example). Quota is five per piece; beyond that, the text should be regenerated rather than rewritten.
- **Semantic fidelity**: before cutting, it builds a document profile (structure map + semantic anchors + argument list). Anchors are never touched; an independent three-way audit runs afterwards and rolls back on drift.
- **Honest boundary**: editing expression cannot cure a lack of facts. Drafts missing first-hand detail get a "facts to supply" list for the author to fill in — authenticity is not a property that can be generated, only supplied by the author.

---

## Directory layout

```
sz-ink/
├── SKILL.md              # Entry: process state machine + routing table + hard constraints
├── rules/
│   ├── metrics.yaml      # Nine metric definitions and per-register thresholds (single source of truth)
│   └── boundaries.md     # Anchor boundaries / experience slots and fabrication limits / whitelist
├── tools/
│   ├── check_metrics.py  # Diagnosis script (nine metrics + --self-test regression)
│   ├── build_profile.py  # Style profile extraction ("write like this person")
│   └── audit_semantic.py # Mechanical check of semantic anchors (numbers, proper nouns, negations preserved)
├── assets/
│   ├── lexicon/          # Cliché / tone / whitespace lexicons + 84 replacement pairs
│   ├── prescription/     # 45 prescription cards across word, sentence, paragraph, structure and perspective layers
│   ├── examples/         # 63 before/after pairs + positive "human-feel" features
│   ├── register/         # Three register profiles (new media / official / explanatory)
│   └── structure/        # 7 hand-written structural skeletons
└── manifest/asset_manifest.json
```

---

## Quick start

```bash
# Diagnose a piece of text (detect register first, then select the threshold set)
python tools/check_metrics.py text.txt --register colloquial_media

# Regression check
python tools/check_metrics.py --self-test
```

Rewriting is driven by the agent following the S0–S8 flow in `SKILL.md`. A human makes the final call; the metrics are only sentinels.

---

## Positioning

sz-ink handles "removing the accent", not "fabricating authenticity": it can bring statistical fingerprints and style back into the human range, but it cannot live your life for you — private details, real prices and the accident that happened that day can only be supplied by the author.

---

## License

MIT

---

## Sources and acknowledgements

- Linguistic basis: publicly known findings from stylistics, lexicology, rhetoric and discourse studies (knowledge is not copyrightable). Quoted characterisations (Wang Zengqi, Yang Jiang, Lao She) and cited scholarship (Shen Jiaxuan, Fang Mei on tail focus) are used as introduction and explanation; sources are annotated item by item inside the assets. No original text by any author is reproduced — all comparison examples are self-written or rewritten and de-identified.
- Lexicon research reference: [blader/humanizer](https://github.com/blader/humanizer) (MIT, Copyright (c) 2025 Siqi Chen) and its Chinese port. Absorbed material has been rewritten and de-identified; only publicly known lexical facts are used.

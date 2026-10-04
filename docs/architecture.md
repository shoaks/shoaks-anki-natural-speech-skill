# Architecture V3

The repository separates specification from implementation responsibilities.

```text
User input
   |
   v
anki-content-generation
   |-- preserve OriginalInput
   |-- produce Target + NaturalMeaning
   |-- produce exactly 2 natural examples
   |-- tokenize examples
   |-- create contextual word/chunk glosses
   |-- extract reusable chunks
   v
schema validation
   |
   v
anki-audio-pipeline
   |-- provider-neutral speech planning
   |-- resolve allowed TTS provider
   |-- synthesize Target audio
   |-- synthesize Example 1 audio
   |-- synthesize Example 2 audio
   |-- validate/cache 3 required artifacts
   v
anki-card-format
   |-- fixed audio-only front
   |-- fixed back hierarchy
   |-- per-word tap gloss interaction contract
   v
anki-deck-builder
   |-- map structured data to Anki fields
   |-- render HTML/CSS/JS
   |-- provide no-JS gloss fallback
   |-- embed 3 required audio files
   |-- export APKG
   |-- verify media map and sound references
   v
Validated APKG
```

## Skill boundaries

### `anki-card-format`
Owns what the learner sees and how the card behaves.
It does not generate linguistic content or audio.

### `anki-content-generation`
Owns Target, meaning, examples, glosses, and reusable chunks.
It does not decide presentation, TTS provider, or packaging.

### `anki-audio-pipeline`
Owns speech planning, provider selection, synthesis, validation, and cache.
It must synthesize the exact text received from the content layer.

### `anki-deck-builder`
Owns rendering and package mechanics.
It may not modify semantic content to make implementation easier.

## Fixed standard-note invariants

- Front is audio-only.
- Back starts with Target then NaturalMeaning.
- Exactly 2 spoken examples.
- Every lexical example word has contextual tap-gloss coverage.
- Useful multi-word units may share one chunk gloss.
- `More` appears after both examples.
- Exactly 3 mandatory natural-audio artifacts: Target + Example 1 + Example 2.
- Required audio must be physically embedded in APKG media.
- All learner-facing explanations are English.

## Failure containment

- No layer may weaken another layer's contract.
- Missing gloss coverage fails before TTS.
- Missing required audio fails before package completion.
- Missing packaged media fails after export.
- Deterministic failures are not retried unchanged.
- Full batch starts only after one representative vertical smoke test passes.

The root `SKILL.md` orchestrates these boundaries and does not redefine them.

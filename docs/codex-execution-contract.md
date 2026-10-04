# Codex Execution Contract V3

## 1. Read order

Before building, read:

1. root `SKILL.md`
2. `skills/anki-card-format/SKILL.md`
3. `skills/anki-content-generation/SKILL.md`
4. `skills/anki-audio-pipeline/SKILL.md`
5. `skills/anki-deck-builder/SKILL.md`
6. `schemas/note.schema.json`
7. active config

Do not infer card rules from old examples or previous exports.

## 2. Preflight

Check runtime, package builder, writable paths, allowed TTS adapters, target-language support, network/paid policy, and model-download policy before any batch generation.

Do not auto-download a large model unless allowed.

## 3. One-note vertical smoke test

Before a batch, run one representative standard note through:

`content -> schema validation -> Target + 2 example TTS -> audio validation -> render -> APKG export -> media verification`

The smoke test passes only if:

- example count is exactly 2;
- both examples have NaturalMeaning;
- all lexical example tokens resolve to contextual glosses;
- Target audio exists and is non-zero;
- Example 1 audio exists and is non-zero;
- Example 2 audio exists and is non-zero;
- front is learner-visible audio only;
- back follows the fixed hierarchy;
- all 3 required audio files have valid `[sound:filename]` references;
- all 3 required audio files are physically present in APKG media;
- all required sound references resolve post-export.

If this fails, stop before the batch.

## 4. Retry discipline

- deterministic unchanged error: 0 retries;
- same provider after deterministic failure: 0 retries;
- transient step: at most 1 retry;
- total provider attempts: bounded by config.

Do not use a full batch as a diagnostic.

## 5. Scope discipline

When fixing a failure, change only the owning layer.

Examples:

- bad meaning/example/gloss -> content skill/layer;
- TTS failure -> audio skill/layer;
- tap interaction or layout -> card-format/deck-builder layer;
- missing packaged media -> deck-builder layer.

Do not alter card semantics to hide an implementation failure.

## 6. Batch behavior

Only after smoke-test success:

- process requested items;
- generate exactly two validated examples per standard note;
- synthesize three required audio artifacts per standard note;
- reuse valid cached audio;
- render the fixed card;
- package once when practical;
- verify every required sound reference after export.

## 7. Completion rule

Do not claim completion unless the APKG has passed final validation.

These are not completion:

- content JSON only;
- planned audio filenames;
- TTS prompts or speech plans;
- external audio files not embedded in APKG;
- an APKG with unresolved media;
- missing tap-gloss coverage;
- a front containing answer text;
- any standard note with an example count other than 2.

## 8. Failure report

On failure, report only:

- failed stage;
- affected Target/example;
- provider if relevant;
- first root error;
- whether a retry occurred;
- the exact external action required only when Codex cannot perform it itself.

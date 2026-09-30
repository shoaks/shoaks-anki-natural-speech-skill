# Codex Execution Contract

This contract exists to prevent repeated failed attempts, unnecessary model downloads, and full-batch rebuilds before the pipeline is proven.

## 1. Read before acting

Before coding or building, read:

1. `SKILL.md`
2. `config.example.yaml`
3. `schemas/note.schema.json`
4. `schemas/speech-plan.schema.json`
5. this file

If they conflict, fix the specification conflict first. Do not implement around contradictory requirements.

## 2. Preflight first

Preflight must determine without starting a full build:

- Python/runtime availability
- `genanki` availability
- registered TTS adapters
- which adapters are actually runnable
- target-language support
- local/network/paid policy
- whether large model downloads are allowed
- writable output/cache paths

Do not auto-install or auto-download a large TTS model unless explicitly allowed.

If no allowed provider is available, stop at preflight and return one concise actionable setup error.

## 3. One-note vertical smoke test

Before processing a batch, run exactly one representative note through the complete path:

`analysis -> speech plan -> provider resolve -> synthesize -> audio validate -> note build -> APKG export -> APKG media verify`

The smoke test passes only if:

- audio synthesis actually ran
- output file exists and is non-zero
- final note records the actual provider
- card contains `[sound:filename]`
- APKG media contains the same file
- APKG media map is non-empty

If the smoke test fails, STOP. Do not start the batch.

## 4. Retry budget

Defaults:

- same deterministic error: 0 retries
- same provider after deterministic failure: 0 retries
- transient step retry: at most 1
- total provider attempts: bounded by config

Examples of deterministic failures:

- missing executable/module
- unsupported language
- invalid configuration
- permission denied
- model absent while auto-download is disabled
- schema mismatch

Do not rerun an unchanged command after a deterministic error.

## 5. Provider behavior

Core code must call the provider abstraction.

Do not:

- hard-code one TTS engine in core build code
- install a provider merely because it appears in an example
- silently switch to a paid/network service
- write a fake audio filename when synthesis failed

When `provider: auto`, discover allowed installed adapters and resolve one at runtime.

## 6. Cache behavior

Cache successful media using a stable key including at least:

- target text
- language/dialect
- provider
- model
- voice
- speech-plan version
- audio mode

Do not regenerate unchanged successful media or images.

## 7. Batch behavior

Only after smoke test passes:

- process the requested batch
- reuse cache
- validate each note
- package once when practical
- run final APKG verification

If one item fails, report the item and stage precisely. Do not rebuild already successful unchanged items unless necessary.

## 8. Scope discipline

When fixing a failure:

- change the smallest relevant layer
- do not rewrite unrelated templates/schemas
- do not change card semantics to hide an implementation failure
- do not relax mandatory validation merely to make the build pass

## 9. Completion language

Never say the deck is complete unless the artifact passed final validation.

These do not count as completion:

- planned filename
- TTS prompt
- speech plan
- placeholder
- JSON field naming a nonexistent file
- audio file outside the APKG
- APKG with empty media for pronounceable notes

## 10. Failure report

On stop, report only:

- failed stage
- exact provider/adapter if relevant
- first root error
- whether a retry was attempted
- what is required to proceed

Do not bury the blocker under repeated logs.

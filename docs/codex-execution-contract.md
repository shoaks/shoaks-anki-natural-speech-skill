# Codex Execution Contract

This contract exists to prevent repeated failed attempts, unnecessary model downloads, incomplete audio, and full-batch rebuilds before the pipeline is proven.

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

## 3. Stop before expensive work

Never use expensive work as a diagnostic step.

Before any operation that may download a model, install a large dependency, synthesize a full batch, regenerate many media files, or rebuild the whole APKG, verify the immediately preceding prerequisite first.

Required progression:

`inspect -> probe -> one-note vertical test -> batch`

Never use:

`guess -> install/download -> retry unchanged -> full batch`

## 4. One-note vertical smoke test

Before processing a batch, run exactly one representative normal note through the complete path:

`analysis -> generate >=4 examples -> speech plans -> provider resolve -> synthesize Target + every example -> audio validate -> note build -> APKG export -> APKG media verify`

The smoke test passes only if:

- the note contains at least 4 valid examples;
- Target audio synthesis actually ran;
- every pronounceable example has its own synthesis run and file;
- every required audio file exists and is non-zero;
- the final note records the actual provider;
- every required audio file has a valid `[sound:filename]` reference;
- APKG media physically contains every required Target/example audio file;
- post-export verification resolves every sound reference;
- the APKG media map is non-empty.

For a representative note with exactly four examples, expect at least five verified natural-audio artifacts.

If the smoke test fails, STOP. Do not start the batch.

## 5. Retry budget

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

## 6. Provider behavior

Core code must call the provider abstraction.

Do not:

- hard-code one TTS engine in core build code
- install a provider merely because it appears in an example
- silently switch to a paid/network service
- write a fake audio filename when synthesis failed
- treat Target audio as sufficient when example audio is missing

When `provider: auto`, discover allowed installed adapters and resolve one at runtime.

## 7. Cache behavior

Cache successful media using a stable key including at least:

- text
- language/dialect
- provider
- model
- voice
- speech-plan version
- audio mode

Target and example audio use the same cache discipline.

Do not regenerate unchanged successful media or images.

## 8. Batch behavior

Only after smoke test passes:

- process the requested batch;
- generate at least 4 valid examples for every normal word/phrase/sentence note;
- synthesize Target and every required example;
- reuse cache;
- validate each note;
- package once when practical;
- run final APKG verification for every required sound reference.

If one item fails, report the item and stage precisely. Do not rebuild already successful unchanged items unless necessary.

## 9. Packaging invariant

Audio existing on disk is not enough.

Every required Target/example audio file must be copied into the APKG media collection and referenced by `[sound:filename]`.

External paths, URLs, cache-only files, and media left outside the APKG are build failures.

## 10. Scope discipline

When fixing a failure:

- change the smallest relevant layer;
- do not rewrite unrelated templates/schemas;
- do not change card semantics to hide an implementation failure;
- do not reduce the four-example minimum;
- do not disable example audio;
- do not relax mandatory validation merely to make the build pass.

## 11. Completion language

Never say the deck is complete unless the artifact passed final validation.

These do not count as completion:

- planned filename
- TTS prompt
- speech plan
- placeholder
- JSON field naming a nonexistent file
- external audio path/URL
- audio file outside the APKG
- missing example audio
- fewer than 4 required examples
- APKG with unresolved required media

## 12. Failure report

On stop, report only:

- failed stage
- affected Target/example
- exact provider/adapter if relevant
- first root error
- whether a retry was attempted
- what is required to proceed

Do not bury the blocker under repeated logs.

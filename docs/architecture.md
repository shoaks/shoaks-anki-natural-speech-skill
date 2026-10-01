# Architecture

```text
User input
   |
   v
Input parser
   |-- preserve exact OriginalInput
   |-- classify word / phrase / sentence
   v
Target normalizer
   |-- infer intended meaning
   |-- produce one correct natural Target
   |-- record correction separately
   v
Linguistic analyzer
   |-- NaturalMeaning
   |-- Context
   |-- Chunks
   |-- Grammar
   |-- Vocabulary
   |-- Usage
   |-- Examples[]
   |-- word / phrase / sentence => >=4 Examples required
   v
Speech planner (provider-neutral)
   |-- build plan for Target
   |-- build plan for every Example.Text
   |-- Pronunciation / NaturalSpeech
   |-- stress / focus / rhythm / linking / reductions / intonation
   v
TTS provider registry
   |-- discover registered adapters
   |-- cheap availability probe
   |-- filter by language + policy
   |-- provider: auto or explicit
   v
TTS provider adapter
   |-- mandatory Target AudioNatural
   |-- mandatory per-example AudioNatural
   |-- actual resolved provider/model
   |-- optional AudioCareful
   |-- configurable fallback chain
   v
Image adapter
   |-- semantic image when useful
   v
Anki note builder
   |-- Listening card (default; audio-first, Target hidden on front)
   |-- Comprehension card (optional)
   |-- Production card (selective)
   |-- Error Correction card (selective)
   v
Audio validation
   |-- every Target/example file exists / non-zero
   |-- every required sound reference exists
   v
APKG packaging
   |-- physically embed every required Target/example audio file
   v
Post-export validation
   |-- >=4 examples for normal notes
   |-- resolve every [sound:filename] to APKG media
   |-- ambiguous production-card gate
   v
APKG / CSV + media export
```

## Boundary rules

- `OriginalInput` preserves what the learner typed.
- `Target` is the one correct natural item to learn.
- The default front is audio-first: Target text remains hidden until reveal.
- Every normal word/phrase/sentence note requires at least 4 complete natural examples.
- Linguistic analysis decides meaning and reusable structure.
- Speech planning decides how Target and each example are naturally realized.
- The selected TTS provider realizes the plan and must not invent grammar or meaning.
- Target audio and every example audio are mandatory when pronounceable.
- Image generation illustrates semantics and must not reveal the written answer.
- Missing valid Target audio is a build error.
- Missing any required example audio is a build error.
- Audio outside the APKG is not sufficient; required media must be physically packaged.
- Fewer than 4 examples for a normal note is a build error.
- Production cards are generated only when the cue constrains the answer sufficiently.
- Paid services must never be silently selected.

## Failure containment

- Core build code must not import a specific TTS engine.
- Provider-specific code lives behind an adapter.
- `provider: auto` does not trigger large automatic downloads by default.
- Full-batch generation starts only after a one-note vertical smoke test passes with Target + all required example audio.
- A generated audio file that is not referenced and packaged into the APKG is still a failure.
- An unchanged deterministic failure must not be retried.

See `docs/codex-execution-contract.md`.

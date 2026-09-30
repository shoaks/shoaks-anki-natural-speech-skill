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
   |-- Example
   |-- single word => Example is mandatory
   v
Speech planner (provider-neutral)
   |-- Pronunciation
   |-- NaturalSpeech
   |-- stress / focus
   |-- rhythm / linking / reductions
   |-- intonation
   v
TTS provider registry
   |-- discover registered adapters
   |-- cheap availability probe
   |-- filter by language + policy
   |-- provider: auto or explicit
   v
TTS provider adapter
   |-- actual resolved provider/model
   |-- mandatory AudioNatural
   |-- optional AudioCareful
   |-- configurable fallback chain
   v
Image adapter
   |-- semantic image when useful
   v
Anki note builder
   |-- Comprehension card (default)
   |-- Listening card (conditional)
   |-- Production card (selective)
   |-- Error Correction card (selective)
   v
Audio validation
   |-- exists / non-zero / decodable when practical
   v
Validation
   |-- audio/media gate
   |-- single-word example gate
   |-- ambiguous production-card gate
   v
APKG / CSV + media export
```

## Boundary rules

- `OriginalInput` preserves what the learner typed.
- `Target` is the correct, natural item to learn.
- Linguistic analysis decides meaning and reusable structure.
- Every lexical single-word note must contain at least one natural complete example sentence.
- Speech planning decides how the target is naturally realized in connected speech.
- The selected TTS provider realizes the plan and must not invent grammar or meaning.
- Image generation illustrates semantics and must not reveal the written answer.
- Missing valid TTS audio is a build error.
- Missing single-word `Example` is a build error.
- Production cards are generated only when the cue constrains the answer sufficiently.
- Paid services must never be silently selected.

## Failure containment

- Core build code must not import a specific TTS engine.
- Provider-specific code lives behind an adapter.
- `provider: auto` does not trigger large automatic downloads by default.
- Full-batch generation starts only after a one-note vertical smoke test passes.
- A generated audio file that is not referenced and packaged into the APKG is still a failure.
- An unchanged deterministic failure must not be retried.

See `docs/codex-execution-contract.md`.
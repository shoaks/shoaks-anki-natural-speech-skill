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
Speech planner
   |-- Pronunciation
   |-- NaturalSpeech
   |-- stress / focus
   |-- rhythm / linking / reductions
   |-- intonation
   v
TTS provider adapter
   |-- provider: auto or explicit
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

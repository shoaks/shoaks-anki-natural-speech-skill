# Architecture

```text
User input
   |
   v
Input parser
   |
   v
Linguistic analyzer
   |-- EnglishMeaning
   |-- Segmentation
   |-- Grammar
   |-- Vocabulary
   |-- Usage
   |
   v
Speech planner
   |-- SpokenForm
   |-- stress / focus
   |-- rhythm
   |-- linking / reductions
   |-- intonation
   |
   v
Local TTS adapter
   |-- Kokoro
   |-- Fish Speech
   |-- optional fallback
   |
   +------> AudioNatural
   +------> AudioCareful (optional)
   |
   v
Image adapter
   |
   v
Anki note builder
   |
   +-- Recognition card
   +-- Cloze card (selective)
   +-- Production card (selective)
   |
   v
Validation
   |
   v
APKG / CSV + media export
```

## Boundary rules

- Linguistic analysis decides what the sentence means.
- Speech planning decides how the same sentence can naturally be realized in speech.
- TTS realizes the speech plan; it must not invent grammar or meaning.
- The displayed target remains standard written language unless the user's source itself is nonstandard.
- Paid services are optional adapters only and must never be silently selected.

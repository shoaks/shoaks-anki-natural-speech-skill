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
Fish Speech TTS
   |-- mandatory AudioNatural synthesis
   |-- optional AudioCareful synthesis
   |-- no automatic TTS fallback
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
- Fish Speech is mandatory for exported natural audio.
- Missing or failed Fish Speech synthesis is a build error; do not export a silent deck.
- Paid services must never be silently selected.

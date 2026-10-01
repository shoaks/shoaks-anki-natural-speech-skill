# Card Design V2

This document defines the fixed Anki card layout and content hierarchy.

## Canonical fields

1. Target
2. NaturalMeaning
3. Context
4. Chunks
5. Grammar
6. Vocabulary
7. Usage
8. Pronunciation
9. NaturalSpeech
10. Examples
11. OriginalInput
12. CorrectionNote
13. AudioNatural
14. Image
15. Tags

Each `Examples[]` item contains at least:

- `Text`
- `AudioNatural`

An optional English `NaturalMeaning` may be added to an example when useful.

## Core learning sequence

`sound -> retrieval -> reveal -> meaning -> reusable pattern -> spoken transfer examples`

The learner should hear first and reveal the written answer second.

## Card A — Listening — default

Generate for every useful pronounceable note.

### Front

```text
┌──────────────────────────────┐
│ FRONT                        │
│                              │
│ 🔊 AudioNatural              │
│                              │
│ [optional semantic image]    │
│                              │
│ Target is NOT shown          │
└──────────────────────────────┘
```

Rules:

- Audio is the primary cue.
- Do not display Target or a transcription before reveal.
- Do not show English meaning.
- Image is optional and must not reveal the written answer.
- Target audio is mandatory.

### Back

```text
┌──────────────────────────────┐
│ BACK                         │
│                              │
│ Target                       │
│                              │
│ NaturalMeaning               │
│                              │
│ key chunk / pattern          │
│                              │
│ NaturalSpeech information    │
│                              │
│ EXAMPLES                     │
│ 1. sentence 🔊              │
│ 2. sentence 🔊              │
│ 3. sentence 🔊              │
│ 4. sentence 🔊              │
└──────────────────────────────┘
```

Back order:

1. Target
2. NaturalMeaning
3. key Chunk / Grammar pattern
4. Pronunciation / NaturalSpeech when useful
5. at least 4 Examples, each with its own playable audio
6. optional Vocabulary / Usage
7. OriginalInput / CorrectionNote when relevant

The back may contain more analysis, but it must not bury the core answer under low-value detail.

## Card B — Comprehension — optional

Generate only when seeing the written Target first trains a genuinely different skill.

Front may contain:

- Target
- optional semantic image
- optional Target audio replay

Back:

- NaturalMeaning
- key pattern
- selected usage detail

Do not generate this card automatically merely because Target text exists.

## Card C — Production — selective

Generate only when the cue sufficiently constrains the answer.

Front:

- contextual English cue
- optional semantic image
- optional target-language construction hint

Back:

- Target
- AudioNatural
- key pattern

Do not create production cards when many target-language answers are equally valid.

## Optional Card D — Personal Error Correction

Generate only for a real, useful, likely-to-recur learner error.

The front may show the original learner error. The back shows the corrected form plus one short English explanation.

Correct input must never produce a fake correction task.

## Examples — hard requirement

Every normal `word`, `phrase`, and `sentence` note requires **at least 4 complete natural target-language example sentences**.

Three or fewer examples is invalid.

Each example must:

1. be natural;
2. be simple enough to keep the target pattern visible;
3. be semantically faithful;
4. add transfer value;
5. represent a plausible common/current use;
6. have its own real generated natural-speech audio file.

Do not meet the minimum with trivial near-duplicates.

For a single word, the four examples should collectively demonstrate useful collocations, constructions, argument structure, grammatical behavior, or contextual variation.

## Example audio — hard requirement

Every pronounceable example must have a distinct `AudioNatural` artifact.

Example audio is required even when the example is not exported as a standalone card.

For a normal note with exactly four examples, the minimum natural-audio payload is:

```text
1 Target audio
4 Example audio files
= 5 required audio files
```

Every required file must:

- actually be synthesized;
- exist;
- be non-empty;
- be referenced by `[sound:filename]`;
- be physically embedded in APKG media;
- resolve during post-export package inspection.

External paths, URLs, cache-only files, placeholders, and filename strings do not count.

## Single-word design

A single-word note is not a dictionary card.

Required learning path:

`word sound -> retrieve word -> reveal meaning/pattern -> 4+ spoken examples -> transfer`

### Nouns

Show article/gender when relevant and useful verbs, prepositions, or collocations across the examples.

### Verbs

Show argument structure or common constructions across the examples.

### Adjectives

Show agreement or natural noun/copular collocations.

### Adverbs / conjunctions

Show normal sentence position or discourse function.

## Typography

Target:

```css
.target {
  font-size: clamp(28px, 5vw, 34px);
  font-weight: 600;
  line-height: 1.35;
}
```

Body:

```css
.card-content {
  max-width: 680px;
  margin: 0 auto;
  font-size: 17px;
  line-height: 1.55;
}
```

## Light / dark mode

Do not hard-code a white or cream background.

Use CSS variables:

```css
:root {
  --bg: #fafafa;
  --text: #202124;
  --secondary: #666;
  --border: #e3e3e3;
  --accent: #3f6fb5;
}

.nightMode {
  --bg: #181818;
  --text: #eeeeee;
  --secondary: #aaaaaa;
  --border: #333333;
  --accent: #8ab4f8;
}
```

## Content limits

- NaturalMeaning: usually 1–2 short lines
- Chunks: usually 1–4
- Grammar: normally 1 core transferable pattern
- Vocabulary: 0–5 useful items
- Usage: 0–3 short notes
- Examples: minimum 4 for normal word/phrase/sentence notes; add more only when they add distinct transfer value
- Pronunciation: only when useful

## Forbidden patterns

- Do not reveal Target text on the default Listening card front.
- Do not create `beautiful -> bellas` without enough context.
- Do not create `divertido / divertida` as one Target.
- Do not combine two alternative answers into one Target.
- Do not repeatedly present an incorrect source form as the main recognition stimulus.
- Do not use three or fewer examples for a normal word/phrase/sentence note.
- Do not leave any required example without real audio.
- Do not claim image/audio completion unless actual media is packaged in the APKG.
- Do not satisfy audio requirements with external file paths or URLs.
- Do not export an APKG whose required Target/example sound references cannot be resolved to packaged media.

## Recommended generation policy

Listening is the default card for each useful pronounceable note.

Comprehension, Production, and Error Correction are conditional. They are not quotas. Every extra card must train a distinct retrieval skill.

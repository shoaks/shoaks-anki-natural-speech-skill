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
10. Example
11. OriginalInput
12. CorrectionNote
13. AudioNatural
14. Image
15. Tags

## Core learning sequence

`target language -> comprehension -> sound -> reusable pattern -> transfer`

The default card must expose correct target-language input, not the learner's mistake.

## Card A — Comprehension

### Front

```text
┌────────────────────────────┐
│          AUDIO             │
│                            │
│          TARGET            │
│                            │
│           IMAGE            │
└────────────────────────────┘
```

Rules:

- Target is visually dominant.
- Do not show English meaning.
- Image is optional when visual meaning is weak.
- Audio is mandatory for pronounceable targets.

### Back

Use this order:

1. Target + audio
2. NaturalMeaning
3. Chunks
4. Grammar
5. Vocabulary
6. Usage
7. Pronunciation / NaturalSpeech
8. Example
9. OriginalInput / CorrectionNote

Information priority:

`Meaning -> Chunk -> Grammar -> Vocabulary -> Usage -> Pronunciation -> Example -> Error history`

## Card B — Listening

Front: AudioNatural only; optional semantic image.

Do not display target-language text initially.

Back:

- Target
- NaturalMeaning
- key Chunk
- Pronunciation / NaturalSpeech

This card trains `sound -> recognition -> meaning`.

## Card C — Production

Generate selectively.

Front:

- contextual English cue
- optional semantic image
- optional target-language keyword/construction hint

Back:

- Target
- AudioNatural
- key pattern

Do not create production cards when many target-language answers are equally valid.

## Optional Card D — Personal Error Correction

Generate only for real, useful, likely-to-recur errors.

The front may show the original learner error. The back shows the corrected form plus one short English explanation.

Correct input must never produce a fake correction task.

## Single-word design — mandatory example

A single-word note is not a dictionary card.

Required learning path:

`word -> core meaning -> collocation/construction -> natural sentence -> sound -> image/context`

Every lexical single-word note must contain at least one complete natural target-language example sentence.

Normal phrase and sentence notes also require at least one new transfer example sentence. Only items explicitly classified as `other` may omit it when sentence use is genuinely inappropriate.

### Nouns

Show article/gender when relevant and a common verb, preposition, or collocation.

Example:

```text
costa
the coast

en la costa

Pasamos una semana en la costa.
```

### Verbs

Show argument structure or a common construction.

Example:

```text
aprovechar
to make use of; take advantage of

aprovechar algo
aprovechar para + infinitive

Voy a aprovechar el fin de semana para descansar.
```

### Adjectives

Show agreement or a natural noun/copular collocation.

### Adverbs / conjunctions

Show normal sentence position or discourse function.

## Example quality gate

The example must be:

1. natural;
2. simple;
3. semantically faithful;
4. useful for transfer;
5. representative of a common/current use.

Reject low-value filler such as:

`Las legumbres son buenas.`

Prefer:

`Como legumbres dos o tres veces por semana.`

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
- Example: at least 1 complete natural sentence for normal word, phrase, and sentence notes
- Pronunciation: only when useful

## Forbidden patterns

- Do not create `beautiful -> bellas` without enough context.
- Do not create `divertido / divertida` as one Target.
- Do not combine two alternative sentences into one Target.
- Do not repeatedly present an incorrect source form as the main recognition stimulus.
- Do not claim image/audio completion unless the actual media is packaged in the APKG.
- Do not export an APKG with an empty media map when pronounceable notes require audio.

## Recommended generation policy

For about 100 notes, a typical result may be:

- 100 Comprehension
- 60–90 Listening
- 20–40 Production
- 10–30 Error Correction

These are not quotas. Every extra card must train a distinct retrieval skill.

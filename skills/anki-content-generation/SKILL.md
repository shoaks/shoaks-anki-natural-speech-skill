---
name: anki-content-generation
summary: Generate structured linguistic content for the fixed audio-first Anki card format.
---

# Anki Content Generation Skill

## Scope

This skill owns linguistic content only. It MUST NOT redesign the card, select a TTS engine, synthesize audio, or package APKG files.

It produces the structured content consumed by the card-format and build skills.

## Input preservation

- Preserve the learner's exact source in `OriginalInput`.
- If the source is clearly wrong, create one corrected natural `Target` and an English `CorrectionNote`.
- Never silently overwrite the source.
- Never merge competing alternatives into one Target.

## Language policy

Target-language material remains in the target language.
All learner-facing explanations are English only.

## NaturalMeaning

Use natural contextual English, not mechanical word-for-word translation.

## Examples

Generate exactly TWO complete natural examples for every normal word, phrase, or sentence note.

Each example MUST:

- represent plausible current native usage;
- reinforce the Target word, phrase, construction, or communicative function;
- add transfer value;
- differ meaningfully from the other example;
- stay simple enough that the relevant pattern remains visible;
- have a natural English meaning;
- be suitable for spoken audio generation.

Reject filler examples and trivial near-duplicates.

## Example token data

Each example MUST contain:

- `Text`
- `NaturalMeaning`
- `Tokens[]`
- `Glosses[]`

Every lexical token MUST have a `GlossID` that resolves to one contextual gloss. Punctuation uses `GlossID: null`.

Token shape:

```json
{
  "Text": "pensar",
  "GlossID": "g2",
  "IsPunctuation": false
}
```

Gloss shape:

```json
{
  "ID": "g2",
  "Meaning": "to think",
  "Lemma": "pensar",
  "PartOfSpeech": "verb",
  "Note": "infinitive"
}
```

## Chunk-first glossing

Do not mechanically teach every token as an independent dictionary item.

When a multi-word unit is the better learning unit, multiple tappable word tokens SHOULD resolve to the same chunk gloss.

Example: the tokens `un` and `momento` may both resolve to a gloss for `un momento` = `a moment`, with a note that it is a common conversational chunk.

Prioritize learning units in this order:

1. constructions
2. collocations and chunks
3. lexical words
4. morphology when useful

Glosses explain the meaning HERE, not every dictionary sense.

## Chunks

Extract only reusable patterns with transfer value, for example:

- `déjame + infinitive` = `let me do something`
- `tener que + infinitive` = `to have to do something`

## Optional secondary notes

Grammar, usage, pronunciation, and natural-speech notes are optional. Include them only when they materially improve comprehension or transfer. Keep them concise because they belong in the secondary `More` section.

## Single-word input

A single word is not a dictionary-card task. Generate the Target, contextual meaning, useful grammatical information when relevant, a useful collocation/construction when available, exactly two natural examples, and full contextual gloss coverage for both examples.

## Validation

Content is invalid if any of these are true:

- Target is empty;
- NaturalMeaning is empty or unnatural;
- learner-facing explanations are not English;
- example count is not exactly 2;
- either example is incomplete, unnatural, or lacks NaturalMeaning;
- a lexical token does not resolve to a gloss;
- glosses are dictionary dumps rather than contextual meanings;
- examples are trivial near-duplicates;
- useful chunks are split into misleading token-by-token equivalences.

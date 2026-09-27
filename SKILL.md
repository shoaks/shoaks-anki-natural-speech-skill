---
name: anki-natural-speech
summary: Build language-learning Anki decks from user-provided words and sentences using English-only explanations, semantic images, and local natural-speech TTS.
---

# Anki Natural Speech Skill

## Goal

Turn user-provided target-language words, phrases, and sentences into high-quality Anki notes and exportable decks.

The skill is optimized for contextual language acquisition rather than dictionary memorization. It must preserve the learner's original target-language text, explain it in English, generate a semantically relevant image, and create native-like conversational audio with local TTS models.

## Non-negotiable rules

1. Preserve the original target-language text in `Target`.
2. All learner-facing explanations and definitions must be in English only.
3. Do not use Chinese in `EnglishMeaning`, `Grammar`, `Vocabulary`, `Usage`, `PronunciationNotes`, or examples' explanations.
4. Prefer contextual meaning over exhaustive dictionary definitions.
5. Treat phrases and grammatical constructions as learning units.
6. Use local models by default. Paid cloud APIs must never be required or silently invoked.
7. Natural audio should sound like ordinary native conversation, not like slow textbook dictation.
8. Never create "human-like" audio by randomly dropping sounds. Reductions, linking, contractions, assimilation, elision, rhythm, and stress must be linguistically plausible.
9. Keep written form, spoken realization, and generated audio as separate layers.
10. One note may generate multiple Anki cards; do not duplicate the linguistic analysis across separate notes.

## Inputs

Accept:

- a single word
- a phrase
- one sentence
- multiple sentences
- a vocabulary list
- TXT or CSV input
- short passages that can be split into learning units

Automatically detect the target language when reliable. If uncertain, preserve the source and report `Language: unknown` instead of guessing.

## Core note fields

Each note should support:

- `ID`
- `Target`
- `EnglishMeaning`
- `Image`
- `AudioNatural`
- `AudioCareful`
- `Segmentation`
- `Grammar`
- `Vocabulary`
- `Usage`
- `PronunciationNotes`
- `SpokenForm`
- `Example`
- `Cloze`
- `Tags`
- `Source`
- `Language`
- `Difficulty`
- `ImageMode`

Use stable IDs so regenerated decks can reuse media and update existing notes predictably.

## Linguistic analysis

### EnglishMeaning

Give a natural English meaning. Do not force a literal word-for-word translation when it sounds unnatural.

### Segmentation

Split sentences into syntactic and semantic chunks, not isolated tokens.

Example:

`[Se los] [voy a devolver] [la semana que viene]`

### Grammar

Explain only grammar that contributes meaning or is useful to the learner.

For important structures, prefer:

- **Form**
- **Function**
- **Meaning in this sentence**

Do not repeatedly explain trivial articles or prepositions unless they matter in context.

### Vocabulary

For important lexical items provide:

- lemma
- part of speech
- contextual English definition
- grammatical gender when relevant
- useful irregular form when relevant
- common collocation or construction when useful

Do not dump every dictionary sense.

### Usage

Add concise notes only when useful, such as:

- formal vs informal
- spoken vs written
- regional variation
- idiomatic meaning
- common collocations
- natural alternatives

## Image generation

Generate one semantic image when a visual can genuinely reinforce meaning.

Image requirements:

- no text
- no subtitles
- no vocabulary labels
- no translations
- no answer embedded in the image
- simple composition
- one clear semantic concept or event
- avoid irrelevant decorative detail

For sentence cards, represent the whole event when practical rather than illustrating one noun only.

Use `ImageMode`:

- `generate`
- `optional`
- `skip`

Use `skip` for concepts where an image would be misleading, such as many articles, particles, abstract conjunctions, or grammar-only structures.

## Local TTS architecture

Use a replaceable provider interface. Do not hard-code the deck builder to one model.

Preferred providers:

- English: Kokoro when quality is sufficient
- Multilingual / Spanish / Portuguese: Fish Speech
- Expressive conversational mode: Fish Speech
- Optional local fallback: XTTS or another configured local model

Conceptual interface:

```python
generate_speech(
    text,
    language,
    voice,
    speech_plan,
    mode,
    output_path,
)
```

Normal operation must not require a paid API key.

## Mandatory speech-planning stage

Sentence audio must use this pipeline:

`Written sentence -> Speech planner -> Spoken realization -> Prosody plan -> Local TTS -> Audio`

The speech planner should determine, when relevant:

- language and dialect
- register
- sentence type
- conversational speech rate
- phrase boundaries and pauses
- sentence stress
- focus words
- intonation contour
- connected speech
- linking
- weak forms
- contractions
- assimilation
- elision
- vowel reduction
- rhythm
- emotional stance when clearly implied by context

Do not fabricate accent features or nonstandard spellings just to make the result sound less robotic.

## WrittenForm vs SpokenForm vs AudioRealization

Always distinguish these concepts.

Example:

Written form:

`What are you going to do?`

Possible conversational spoken form:

`What're you gonna do?`

Audio realization:

A natural native-like question with connected speech, reduced unstressed material, plausible stress, and ordinary conversational intonation.

`SpokenForm` is supporting metadata. It must never overwrite `Target`.

Only generate a distinct `SpokenForm` when there is a meaningful and linguistically defensible difference from the written form.

## Natural audio

`AudioNatural` is the primary track.

Target characteristics:

- native-like pronunciation
- ordinary conversational speed
- natural rhythm
- natural sentence stress
- connected speech
- plausible reductions
- realistic phrase-level pauses
- intonation appropriate to sentence type

Avoid:

- equal stress on every word
- artificial pauses between words
- exaggerated acting
- deliberately unclear pronunciation
- random slurring or deletion

The objective is not "less clear TTS". The objective is linguistically authentic conversational speech.

## Careful audio

`AudioCareful` is optional.

Use it for difficult sentences or when the learner enables pronunciation study mode.

Characteristics:

- somewhat slower than natural speech
- clearly articulated
- standard pronunciation
- still rhythmically natural
- never syllable-by-syllable unless explicitly requested

Do not automatically generate two audio files for every easy card unless configured to do so.

## Pronunciation notes

Add notes only when they materially help the learner, for example:

- a common weak form
- a contraction
- a linking pattern
- an unusual stress pattern
- a reduced vowel
- a common elision
- an important regional pronunciation difference

Keep them concise.

## Card generation

Create one rich note, then derive cards from it.

### Card A: Recognition

Front:

- Target
- Image
- Natural audio control

Back:

- EnglishMeaning
- Segmentation
- Grammar
- Vocabulary
- Usage
- PronunciationNotes
- optional Careful audio

### Card B: Cloze

Create only when a useful lexical item or construction is worth testing.

Each cloze should test one meaningful retrieval target. Do not blank multiple unrelated elements in one cloze.

### Card C: Production

Optional. Use for high-value phrases, constructions, vocabulary, or common sentences.

Front:

- Image
- EnglishMeaning

Back:

- Target
- Natural audio
- PronunciationNotes

Do not automatically create production cards for every source item; excessive reverse cards increase review load.

## Single-word input

For a single word include, when relevant:

- lemma
- part of speech
- contextual English definition
- morphology
- grammatical gender
- common collocations
- one natural target-language example
- image
- local natural audio

Do not add full conjugation or declension tables unless explicitly requested.

## Tags

Generate hierarchical tags when useful, for example:

- `Spanish::A1`
- `Topic::DailyLife`
- `Grammar::ObjectPronouns`
- `Grammar::NearFuture`
- `Vocabulary::CommonVerb`
- `Pronunciation::ConnectedSpeech`
- `Source::UserInput`

Do not claim an official CEFR level unless reliable evidence supports it. A local heuristic difficulty tag is acceptable if clearly treated as approximate.

## Local-first configuration

Default policy:

```yaml
tts:
  mode: local
  providers:
    english: kokoro
    multilingual: fish
    expressive: fish
  default_audio: natural
  generate_careful_audio: auto
  fallback:
    enabled: true

image:
  mode: local_preferred

cost_policy:
  prefer_free: true
  allow_paid_api: false
```

If the preferred model is unavailable:

1. Try another configured local provider that supports the language.
2. If no suitable local TTS exists, generate the card without audio and clearly report the missing dependency.
3. Never silently fall back to a paid service.

## Media caching

Do not regenerate identical media unnecessarily.

Use a stable cache key based on at least:

- text
- language
- voice
- audio mode
- speech-plan version
- TTS provider
- model version

A SHA-256 hash is suitable.

## Export

Preferred output:

- `deck.apkg`

Also support:

- UTF-8 CSV
- `media/images/`
- `media/audio/`
- `metadata.json`

Use stable media names such as:

- `anki_000001_image.webp`
- `anki_000001_natural.wav`
- `anki_000001_careful.wav`

Do not use complete sentences as filenames.

## Validation before export

Check every note:

### Text

- `Target` matches the intended source.
- Learner-facing explanations contain no Chinese.
- English meaning is natural.
- Segmentation follows syntax and meaning.
- Grammar explanation matches the actual sentence.
- Vocabulary definitions reflect contextual meaning.

### Image

- Image matches the intended meaning.
- Image contains no answer text.
- Image does not introduce misleading semantic details.

### Audio

- Audio language matches the target.
- Audio is semantically faithful to the source.
- Reductions or omissions are legitimate spoken-language phenomena.
- Speech rate is appropriate.
- Prosody fits the sentence type.
- Stress is plausible.
- There is no random artificial slurring.

## Recommended V1 stack

Keep V1 deliberately simple:

- Anki package generation: `genanki`
- English local TTS: Kokoro
- Multilingual / Spanish / Portuguese local TTS: Fish Speech
- Primary audio: natural conversational mode
- Secondary audio: optional careful mode
- Image generation: local image-generation adapter when configured
- Linguistic analysis: configured LLM that returns English-only structured fields
- Architecture: provider-based and replaceable

Do not add voice cloning in V1. First make the complete pipeline reliable:

`Input -> linguistic analysis -> speech planning -> local TTS -> image generation -> Anki note generation -> APKG export -> validation`

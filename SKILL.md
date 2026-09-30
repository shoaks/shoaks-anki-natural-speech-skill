---
name: anki-natural-speech
summary: Build language-learning Anki decks from user-provided words and sentences using English-only explanations, semantic images, and local natural-speech TTS.
---

# Anki Natural Speech Skill

## Goal

Turn user-provided target-language words, phrases, and sentences into high-quality Anki notes and exportable decks.

The skill is optimized for contextual language acquisition rather than dictionary memorization. It must preserve the learner's original target-language text, explain it in English, generate a semantically relevant image, and create native-like conversational audio with local TTS models.

## Non-negotiable rules

1. Preserve the user's original target-language text exactly in `OriginalInput`.
2. Put the corrected, natural learning form in `Target`; never silently overwrite the source.
3. All learner-facing explanations and definitions must be in English only.
3. Do not use Chinese in `EnglishMeaning`, `Grammar`, `Vocabulary`, `Usage`, `PronunciationNotes`, or examples' explanations.
4. Prefer contextual meaning over exhaustive dictionary definitions.
5. Treat phrases, collocations, and grammatical constructions as learning units.
6. For every lexical single-word input, `Example` MUST contain at least one complete, natural target-language example sentence. This is a build requirement, not an optional enrichment.
7. For a single word, the example must demonstrate a useful common collocation, argument structure, grammatical behavior, or contextual use; reject trivial filler examples.
6. Every pronounceable Target MUST have generated audio. Audio is a required deliverable, not an optional enhancement.
7. Fish Speech is the mandatory TTS engine for `AudioNatural` in this skill. Do not substitute Kokoro, browser TTS, system TTS, cloud TTS, placeholder files, or silent media.
8. Paid cloud APIs must never be required or silently invoked.
9. Natural audio should sound like ordinary native conversation, not like slow textbook dictation.
10. Never create "human-like" audio by randomly dropping sounds. Reductions, linking, contractions, assimilation, elision, rhythm, and stress must be linguistically plausible.
11. Keep written form, spoken realization, and generated audio as separate layers.
12. One note may generate multiple Anki cards; do not duplicate the linguistic analysis across separate notes.

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

Use these learner-facing fields in this order:

1. `Target`
2. `NaturalMeaning`
3. `Context`
4. `Chunks`
5. `Grammar`
6. `Vocabulary`
7. `Usage`
8. `Pronunciation`
9. `NaturalSpeech`
10. `Example`
11. `OriginalInput`
12. `CorrectionNote`
13. `AudioNatural`
14. `Image`
15. `Tags`

Implementation metadata may additionally include `ID`, `Language`, `Dialect`, `AudioCareful`, `Source`, `Difficulty`, `ImageMode`, and `CardPolicy`.

Use stable IDs so regenerated decks can reuse media and update existing notes predictably.

`OriginalInput` preserves what the learner typed. `Target` is the correct, natural form to learn.

## Linguistic analysis

### NaturalMeaning

Give a natural English meaning. Do not force a literal word-for-word translation when it sounds unnatural.

### Chunks

Split material into syntactic and semantic chunks, collocations, and constructions, not isolated tokens.

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

## Mandatory Fish Speech audio

Fish Speech is not a preferred provider; it is a required runtime dependency for audio generation.

For every pronounceable `Target`:

1. Build the speech plan.
2. Generate `AudioNatural` with Fish Speech.
3. Verify that the produced audio file exists, is non-empty, and is referenced by the Anki note.
4. Only then may the note pass validation and be exported.

This applies to:

- single words
- phrases
- full sentences
- generated example sentences when they are exported as separate cards

Do not use Kokoro or another TTS engine as an automatic substitute.

Do not create a successful deck with missing audio.

If Fish Speech is unavailable, missing, fails to initialize, cannot synthesize the target language, or returns an invalid audio file:

**fail the build with a clear actionable error.**

The correct failure mode is:

`Fish Speech unavailable -> build fails -> report setup/runtime error`

The following behavior is forbidden:

`Fish Speech unavailable -> skip audio -> still export deck`

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

The production implementation of this interface must route `AudioNatural` to Fish Speech.

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

## Card generation — V2

Create one rich note, then derive only cards that test distinct retrieval skills.

### Card A: Comprehension — default

Generate for almost every useful note.

Front:

- `AudioNatural`
- `Target`
- `Image` when useful

Do not show English meaning on the front.

Back order:

1. Target + audio
2. NaturalMeaning
3. Chunks
4. Grammar
5. Vocabulary
6. Usage
7. Pronunciation / NaturalSpeech
8. Example
9. OriginalInput / CorrectionNote

### Card B: Listening — conditional

Generate for sentences and useful spoken chunks when listening recognition adds value.

Front:

- `AudioNatural` only
- optional semantic image

Do not initially display the target text.

Back:

- Target
- NaturalMeaning
- key Chunk
- Pronunciation / NaturalSpeech

### Card C: Production — selective

Generate only when the cue sufficiently constrains the answer and the expression is worth active production.

Front:

- contextual English cue
- optional semantic image
- optional target-language keyword/construction hint

Back:

- Target
- AudioNatural
- key pattern

Do not create ambiguous reverse cards such as `beautiful -> bellas`, where several target-language answers may be correct.

### Optional Card D: Personal Error Correction

Create only when:

1. the learner actually made an error
2. the error is likely to recur
3. the contrast has learning value

Only this dedicated card type may use the incorrect original form as the front stimulus.

Do not generate correction cards for already-correct input.

Do not automatically create cloze cards. Cloze is disabled by default in V2.
## Single-word input — hard contract

A single-word note must not degrade into a dictionary-definition card.

For every lexical single-word input include:

- lemma / `Target`
- part of speech
- contextual English meaning
- morphology or grammatical gender when relevant
- at least one useful collocation or construction when available
- **at least one complete, natural target-language example sentence in `Example`**
- pronunciation information when useful
- mandatory Fish Speech audio for the word
- image when semantically useful

The example sentence is mandatory. Validation must fail when a lexical single-word note has an empty `Example`.

Example-quality rules:

- natural: a native speaker could plausibly say it
- simple: avoid burying the target under unrelated advanced vocabulary
- useful: demonstrate the target's common/current sense or construction
- transferable: teach how the word enters real sentences

Part-of-speech guidance:

- noun: show article/gender when relevant plus a common verb/preposition/collocation
- verb: show argument structure, required preposition, reflexive behavior, or a common construction
- adjective: show agreement or a natural noun/copular collocation
- adverb/conjunction: show normal sentence position or discourse function

Weak example:

`Las legumbres son buenas.`

Better example:

`Como legumbres dos o tres veces por semana.`

Do not add full conjugation or declension tables unless explicitly requested.

If the generated example is exported as its own card, it must also receive valid Fish Speech audio.
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

## Required audio configuration

Default policy:

```yaml
tts:
  mode: local
  provider: fish
  audio_required: true
  default_audio: natural
  generate_careful_audio: auto
  fallback:
    enabled: false

image:
  mode: local_preferred

cost_policy:
  prefer_free: true
  allow_paid_api: false
```

Rules:

1. `provider: fish` is mandatory for `AudioNatural`.
2. `audio_required: true` means no note may be exported without a valid Fish Speech audio file.
3. Automatic fallback to another TTS engine is disabled.
4. Automatic fallback to paid services is forbidden.
5. If Fish Speech cannot generate audio, terminate the build and return a clear error with the failed target and setup/runtime cause.
6. Never downgrade the build to "card without audio".

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

- `AudioNatural` is present for every pronounceable Target.
- The `AudioNatural` file exists and is non-empty.
- `AudioNatural` was generated by Fish Speech.
- The Anki card contains a valid `[sound:...]` reference to that file.
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
- Required local TTS for all supported languages: Fish Speech
- `AudioNatural`: mandatory for every pronounceable Target
- Build behavior on Fish Speech failure: hard failure; do not export a silent deck
- Primary audio: natural conversational mode
- Secondary audio: optional careful mode
- Image generation: local image-generation adapter when configured
- Linguistic analysis: configured LLM that returns English-only structured fields
- Architecture: provider-based and replaceable

Do not add voice cloning in V1. First make the complete pipeline reliable:

`Input -> linguistic analysis -> speech planning -> local TTS -> image generation -> Anki note generation -> APKG export -> validation`


## Audio completion gate

Before reporting the task as complete, verify all of the following for every exported pronounceable note:

- a Fish Speech synthesis call actually ran
- the resulting audio file exists
- file size is greater than zero
- the note's `AudioNatural` field is populated
- the Anki template references the generated media through `[sound:filename]`
- the media file is included in the APKG package

A plan, prompt, speech description, filename string, or `TTS Direction` is **not audio generation**.

Do not claim that audio was generated unless an actual playable media file was produced.

If any audio check fails, the overall generation task is failed and must not be reported as successfully completed.

## Card Design V2 reference

The fixed layout, information hierarchy, single-word layout, content limits, dark-mode requirements, and forbidden card patterns are defined in `docs/card-design-v2.md`.

The learning unit is:

`meaningful language pattern + context + sound + transfer`

not `isolated token + dictionary gloss` and not `mistake + correction`.

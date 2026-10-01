---
name: anki-natural-speech
summary: Build language-learning Anki decks from user-provided words and sentences using English-only explanations, semantic images, and local natural-speech TTS.
---

# Anki Natural Speech Skill

## Source of truth

`SKILL.md` is normative. `config.example.yaml`, schemas, docs, and examples must conform to it.
If these files conflict, fix the specification mismatch before implementation. Do not work around contradictory specifications in code.

## Goal

Turn user-provided target-language words, phrases, and sentences into high-quality Anki notes and exportable decks.

The skill is optimized for contextual language acquisition rather than dictionary memorization. It must preserve the learner's original target-language text, explain it in English, generate a semantically relevant image, and create native-like conversational audio with local TTS models.

## Non-negotiable rules

1. Preserve the user's original target-language text exactly in `OriginalInput`.
2. Put one corrected, natural learning form in `Target`; never silently overwrite the source and never combine competing answers into one `Target`.
3. All learner-facing explanations, definitions, grammar notes, usage notes, pronunciation notes, and example explanations must be in English only.
4. Prefer contextual meaning, reusable chunks, collocations, and constructions over exhaustive dictionary definitions.
5. The default study card is **audio-first listening recognition**. Its front MUST contain `AudioNatural` and may contain a semantic image, but MUST NOT display `Target`, its transcription, or an English meaning before reveal.
6. After reveal, the back MUST show `Target`, `NaturalMeaning`, the key reusable pattern/chunk, and useful `NaturalSpeech` information. Additional analysis may follow only when it adds learning value.
7. Every normal `word`, `phrase`, and `sentence` note MUST contain **at least 4 complete, natural target-language example sentences** in `Examples`. Three or fewer is a validation failure.
8. Every example MUST teach a plausible current use, collocation, construction, argument structure, grammatical behavior, or transfer pattern. Reject filler examples.
9. Every pronounceable `Target` MUST have real generated `AudioNatural`.
10. Every pronounceable example in `Examples` MUST have its **own real generated audio file**. Example audio is mandatory even when the examples are not exported as separate cards.
11. Required audio MUST be physically embedded in the exported APKG media archive. An external path, URL, cache entry, filename string, placeholder, speech plan, or un-packaged audio file does not satisfy the requirement.
12. Every required target/example audio file MUST exist, be non-empty, be referenced through `[sound:filename]` in the note/card data, and be present in the APKG media map before export is considered successful.
13. The TTS engine MUST be configurable. Do not hard-code Fish Speech or any other provider as the only valid engine.
14. Paid or network TTS must never be silently selected when policy forbids it.
15. Natural audio should sound like ordinary native conversation. Reductions, linking, contractions, assimilation, elision, rhythm, stress, and intonation must be linguistically plausible; never create naturalness by random deletion or slurring.
16. Keep written form, spoken realization, and generated audio as separate layers.
17. One note may generate multiple Anki cards, but each extra card must test a genuinely different retrieval skill. Production/reverse cards are selective, not automatic.
18. Incorrect learner input must not be the default long-term recognition stimulus; only a dedicated error-correction card may intentionally show it on the front.
19. A missing required example or any missing required audio is a **hard build failure**. Never downgrade the deck to silent cards or fewer examples just to complete the export.

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
10. `Examples`
11. `OriginalInput`
12. `CorrectionNote`
13. `AudioNatural`
14. `Image`
15. `Tags`

`Examples` is an ordered array. Every example item contains at least:

- `Text`: one complete natural target-language sentence
- `AudioNatural`: the filename of that example's own generated natural-speech audio

An example may additionally include `NaturalMeaning` when an English gloss materially helps.

Implementation metadata may additionally include `ID`, `Language`, `Dialect`, `InputType`, `PartOfSpeech`, `AudioProvider`, `AudioModel`, `AudioCareful`, `Source`, `Difficulty`, `ImageMode`, and `CardPolicy`.

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

### Examples

For every normal `word`, `phrase`, and `sentence` note, `Examples` is required and MUST contain **at least 4** complete natural target-language sentences.
Only items explicitly classified as `other` because sentence use is genuinely inappropriate may omit examples.

Each example must:

- be natural and current;
- be simple enough that the target pattern remains visible;
- be semantically faithful;
- add transfer value rather than merely paraphrasing the Target;
- demonstrate a useful collocation, construction, argument structure, grammatical behavior, discourse use, or variation;
- have its own generated `AudioNatural` file.

Do not satisfy the minimum by producing near-duplicate sentences with only a noun or number swapped unless that variation itself teaches a useful pattern.

For single words, the four examples should collectively show how the word enters real sentences. When useful, vary person, tense, argument structure, collocation, or discourse context without turning the examples into a conjugation table.

Example audio follows the same natural-speech quality rules as Target audio and is mandatory regardless of whether the example is exported as its own card.

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

## Mandatory audio generation with pluggable TTS

Real playable audio is mandatory for the main `Target` and for every pronounceable example in `Examples`. The TTS provider is replaceable.

The skill must expose a provider abstraction instead of coupling the build to one engine.

Conceptual interface:

```python
generate_speech(
    text,
    language,
    voice,
    speech_plan,
    mode,
    provider,
    output_path,
)
```

For each required audio item — first the `Target`, then every example `Text`:

1. Build a provider-neutral speech plan.
2. Resolve an allowed configured TTS provider.
3. Actually synthesize a playable audio file.
4. Verify that the file exists and is non-empty.
5. Record the actual resolved provider/model.
6. Create a valid Anki `[sound:filename]` reference.
7. Add that exact file to the APKG media collection.
8. After APKG export, inspect the package and verify that the media map resolves the same file.
9. Only then may that audio item pass validation.

A note with four required examples therefore requires at least **five verified natural-audio artifacts**: one for the Target plus one for each example.

### Audio must be embedded in the deck

"Generated" is not enough. Required audio must be **inside the exported APKG**.

Forbidden substitutes include:

- an external absolute or relative filesystem path;
- an HTTP/HTTPS URL;
- a cache-only file that was not copied into APKG media;
- a filename recorded in JSON without a real file;
- a zero-byte file;
- a TTS prompt or speech plan;
- a `[sound:...]` reference whose media file is absent from the APKG.

If any required Target or example audio is absent from the package, the build fails.

### Provider selection

The provider may be explicitly selected by the user/configuration or resolved automatically from installed/available providers.

When `provider: auto` is used, choose among allowed available providers based on:

- target-language support;
- successful runtime availability;
- ability to produce valid audio;
- naturalness appropriate to the requested mode;
- configured cost/network policy.

Any compatible engine may be implemented as a provider adapter, but the architecture must not assume that a particular engine exists.

Paid or network TTS must not be silently selected when configuration forbids it.

### Failure behavior

If the selected provider fails and fallback is enabled, try configured fallback providers only within the bounded retry policy.

If no allowed provider can produce **all required audio**:

**fail the build with a clear actionable error.**

Correct failure mode:

`required Target/example TTS unavailable or failed -> build fails -> report provider/runtime error`

Forbidden behavior:

`some TTS fails -> omit example audio or Target audio -> export deck anyway`

The core build code MUST NOT import or depend directly on a specific TTS engine. Provider-specific SDK/code belongs behind a provider adapter.
`provider: auto` must probe already available adapters cheaply and MUST NOT auto-download a large model unless configuration explicitly allows it.
The final note must record the actual resolved provider/model; `AudioProvider` must never remain `auto` in an exported note.

## Mandatory speech-planning stage

Sentence audio must use this pipeline:

`Written Target -> provider-neutral speech plan -> selected TTS adapter -> Audio`

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

## Written form vs NaturalSpeech vs AudioRealization

Always distinguish these concepts.

Example:

Written form:

`What are you going to do?`

Possible conversational spoken form:

`What're you gonna do?`

Audio realization:

A natural native-like question with connected speech, reduced unstressed material, plausible stress, and ordinary conversational intonation.

`NaturalSpeech` is supporting metadata describing connected-speech realization. It must never overwrite `Target`.

Only describe a distinct spoken realization when there is a meaningful and linguistically defensible difference from the written form.

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

## Pronunciation

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

### Card A: Listening — default

Generate for every useful pronounceable note.

Front:

- `AudioNatural`
- optional semantic `Image`

The front MUST NOT display:

- `Target`
- a transcription of `Target`
- `NaturalMeaning`
- chunks/grammar that reveal the exact answer

The learner must first retrieve the spoken form from sound.

Back order:

1. `Target`
2. `NaturalMeaning`
3. key reusable `Chunk` / `Grammar` pattern
4. `Pronunciation` / `NaturalSpeech` when useful
5. `Examples`, each shown with its own playable audio
6. optional additional vocabulary/usage detail
7. `OriginalInput` / `CorrectionNote` when relevant

Canonical text layout:

```text
FRONT
🔊 AudioNatural
[optional semantic image]
(no Target text)

BACK
Target
NaturalMeaning
key chunk / reusable pattern
NaturalSpeech information

Examples
1. Example sentence 🔊
2. Example sentence 🔊
3. Example sentence 🔊
4. Example sentence 🔊
```

### Card B: Comprehension — optional

Generate only when seeing the written form before recall adds a genuinely different retrieval skill.

Front may contain:

- `Target`
- optional semantic image
- optional replay of `AudioNatural`

Back contains the meaning and key pattern.

Do not generate this card merely because the fields exist; the audio-first Listening card remains the default.

### Card C: Production — selective

Generate only when the cue sufficiently constrains the answer and the expression is worth active production.

Front:

- contextual English cue;
- optional semantic image;
- optional target-language keyword/construction hint.

Back:

- `Target`;
- `AudioNatural`;
- key pattern.

Do not create ambiguous reverse cards such as `beautiful -> bellas`, where several target-language answers may be correct.

### Optional Card D: Personal Error Correction

Create only when:

1. the learner actually made an error;
2. the error is likely to recur;
3. the contrast has learning value.

Only this dedicated card type may use the incorrect original form as the front stimulus.

Do not generate correction cards for already-correct input.

Do not automatically create cloze cards. Cloze is disabled by default.

## Single-word input — hard contract

A single-word note must not degrade into a dictionary-definition card.

For every lexical single-word input include:

- lemma / `Target`;
- part of speech;
- contextual English meaning;
- morphology or grammatical gender when relevant;
- at least one useful collocation or construction when available;
- **at least 4 complete, natural target-language example sentences in `Examples`**;
- **one real generated audio file for the Target plus one real generated audio file for every example**;
- pronunciation information when useful;
- image when semantically useful.

Validation must fail when a lexical single-word note has fewer than 4 examples or when any required Target/example audio is missing, empty, unreferenced, or absent from APKG media.

Example-quality rules:

- natural: a native speaker could plausibly say it;
- simple: avoid burying the target under unrelated advanced vocabulary;
- useful: demonstrate a common/current sense or construction;
- transferable: teach how the word enters real sentences;
- varied: the four examples should add distinct context or structural value rather than repeat the same sentence frame mechanically.

Part-of-speech guidance:

- noun: show article/gender when relevant plus common verbs, prepositions, or collocations across the examples;
- verb: show argument structure, required preposition, reflexive behavior, tense/person variation when useful, or common constructions;
- adjective: show agreement or natural noun/copular collocations;
- adverb/conjunction: show normal sentence position or discourse function.

Weak example:

`Las legumbres son buenas.`

Better example:

`Como legumbres dos o tres veces por semana.`

Do not add full conjugation or declension tables unless explicitly requested.

Example audio is part of the note's mandatory media payload even when examples are not exported as standalone cards.

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
  provider: auto
  audio_required: true
  example_audio_required: true
  embed_required_audio_in_apkg: true
  default_audio: natural
  generate_careful_audio: auto
  prefer_local: true
  allow_network: false
  allow_paid_api: false
  fallback:
    enabled: true
    providers: []
```

Rules:

1. `provider: auto` means select an allowed available provider at runtime.
2. A user/config may explicitly choose a provider; no provider is globally mandatory.
3. `audio_required: true` means no pronounceable Target may be exported without valid audio.
4. `example_audio_required: true` means every pronounceable example must also have its own valid audio.
5. `embed_required_audio_in_apkg: true` means every required Target/example audio artifact must be physically present in APKG media; external references do not count.
6. Fallback may be enabled across configured providers.
7. Network or paid providers may only be used when explicitly permitted.
8. If all allowed providers fail for any required audio item, terminate the build with a clear error.
9. Never downgrade the build to a card with missing Target audio, missing example audio, or fewer than 4 required examples.
## Codex fail-fast execution rules

Codex MUST follow `docs/codex-execution-contract.md`.

Before a full batch:

1. Run preflight.
2. Discover allowed available TTS adapters without large automatic downloads.
3. Run exactly one representative note through the complete vertical path.
4. The smoke test must include actual synthesis for the representative Target **and all of its required examples**, audio-file validation for each file, Anki sound references for each file, APKG export, and APKG media verification for every required file.
5. If the smoke test fails, STOP and report the first actionable blocker. Do not continue the batch.
6. Do not rerun an unchanged command after the same deterministic error.
7. Respect bounded retry limits from config.
8. Reuse successful cached media instead of regenerating it.
9. Do not relax mandatory validation merely to make the build pass.

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
- `anki_000001_example_01_natural.wav`
- `anki_000001_example_02_natural.wav`
- `anki_000001_example_03_natural.wav`
- `anki_000001_example_04_natural.wav`
- `anki_000001_careful.wav`

Do not use complete sentences as filenames.

## Validation before export

Check every note:

### Text

- `Target` correctly and naturally expresses the intended meaning.
- Learner-facing explanations contain no Chinese.
- `NaturalMeaning` is natural English.
- `Chunks` follow syntax, meaning, collocation, and construction boundaries.
- Grammar explanation matches the actual sentence.
- Vocabulary definitions reflect contextual meaning.
- `OriginalInput` preserves the learner's source exactly.
- For every normal word/phrase/sentence note, `Examples` contains at least 4 complete, natural, useful, non-trivial sentences.
- The four-or-more examples provide transfer value and are not filler near-duplicates.
- Production cards do not rely on highly ambiguous reverse translation.

### Image

- Image matches the intended meaning.
- Image contains no answer text.
- Image does not introduce misleading semantic details.

### Audio

- `AudioNatural` is present for every pronounceable Target.
- Every pronounceable example has its own `AudioNatural`.
- Every required Target/example audio file exists and is non-empty.
- Every required audio artifact was generated by an allowed configured TTS provider.
- `AudioProvider` records the actual resolved provider, not `auto`.
- The Anki note/card data contains a valid `[sound:...]` reference for every required Target/example audio file.
- Every referenced required audio file is physically embedded in the exported APKG media map; external paths/URLs do not count.
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
- TTS: provider adapter with `provider: auto` or explicit provider selection
- `AudioNatural`: mandatory for every pronounceable Target
- Build behavior when all allowed TTS providers fail: hard failure; do not export a silent deck
- Primary audio: natural conversational mode
- Secondary audio: optional careful mode
- Image generation: local image-generation adapter when configured
- Linguistic analysis: configured LLM that returns English-only structured fields
- Architecture: provider-based and replaceable

Do not couple the implementation to one TTS engine. First make the complete pipeline reliable:

`Input -> linguistic analysis -> speech planning -> TTS provider selection -> audio synthesis -> image generation -> Anki note generation -> APKG export -> validation`

## Audio completion gate

Before reporting the task as complete, verify all of the following for **every exported pronounceable Target and every pronounceable example**:

- a configured TTS synthesis call actually ran;
- the resulting audio file exists;
- file size is greater than zero;
- the appropriate Target/example audio field is populated;
- the Anki note/card references the generated media through `[sound:filename]`;
- the exact media file is physically included in the APKG package;
- post-export inspection resolves every required sound reference to a packaged media entry.

For a normal note with exactly 4 examples, the minimum expected natural-audio count is 5 files: 1 Target + 4 examples.

A plan, prompt, speech description, filename string, URL, external path, or `TTS Direction` is **not audio generation** and is not embedded media.

Do not claim that audio was generated unless actual playable media files were produced and packaged.

If any required Target/example audio check fails, the overall generation task is failed and must not be reported as successfully completed.

After APKG export, inspect the package. Missing required media or an empty APKG media map is a hard failure.

## Card Design V2 reference

The fixed layout, audio-first information hierarchy, example/audio requirements, single-word layout, content limits, dark-mode requirements, and forbidden card patterns are defined in `docs/card-design-v2.md`.

The learning unit is:

`sound -> retrieval -> revealed form/meaning -> reusable pattern -> multiple spoken transfer examples`

not `visible answer -> recognition`, not `isolated token + dictionary gloss`, and not `mistake + correction`.

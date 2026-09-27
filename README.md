# Anki Natural Speech Skill

A local-first Codex skill specification for turning user-provided language material into Anki decks with:

- English-only explanations
- contextual sentence segmentation
- grammar and vocabulary analysis
- semantic images
- mandatory native-like conversational audio generated locally with Fish Speech
- optional second careful-pronunciation track
- recognition, cloze, and production cards

## Design goal

The core distinction is:

**written language != careful pronunciation != natural conversational pronunciation**

The skill therefore uses a speech-planning stage before TTS instead of sending the written sentence directly to a speech engine.

## Recommended V1

- `genanki` for `.apkg` export
- Fish Speech as the required TTS engine for all `AudioNatural` generation
- local image generation through a replaceable adapter
- no paid API dependency by default

## Repository structure

```text
.
├── SKILL.md
├── README.md
├── config.example.yaml
├── docs/
│   └── architecture.md
├── examples/
│   ├── input.txt
│   └── expected-note.json
└── schemas/
    ├── note.schema.json
    └── speech-plan.schema.json
```

## First implementation milestone

Build one complete path before adding advanced features:

1. Parse one sentence.
2. Produce structured English-only linguistic analysis.
3. Produce a speech plan.
4. Generate real playable `AudioNatural` with Fish Speech. If Fish Speech fails, fail the build instead of exporting a silent deck.
5. Generate or attach one semantic image.
6. Build one Anki note and recognition card.
7. Export a valid `.apkg`.
8. Add caching and validation.

Voice cloning, large provider matrices, and advanced pronunciation scoring should wait until the end-to-end path is reliable.


## Mandatory audio contract

Every pronounceable exported note must contain a real Fish Speech-generated audio file.

The build must fail if Fish Speech is unavailable or audio synthesis fails. Generating only a speech plan, TTS prompt, filename, or pronunciation notes does not satisfy this requirement.

Silent APKG output is considered a failed build.

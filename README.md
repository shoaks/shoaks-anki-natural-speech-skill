# Anki Natural Speech Skill

A local-first Codex skill specification for turning user-provided language material into Anki decks with:

- English-only explanations
- contextual sentence segmentation
- grammar and vocabulary analysis
- semantic images
- native-like conversational TTS generated locally
- optional careful-pronunciation audio
- recognition, cloze, and production cards

## Design goal

The core distinction is:

**written language != careful pronunciation != natural conversational pronunciation**

The skill therefore uses a speech-planning stage before TTS instead of sending the written sentence directly to a speech engine.

## Recommended V1

- `genanki` for `.apkg` export
- Kokoro for lightweight English TTS
- Fish Speech for multilingual and expressive local TTS
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
4. Generate natural audio with a local TTS provider.
5. Generate or attach one semantic image.
6. Build one Anki note and recognition card.
7. Export a valid `.apkg`.
8. Add caching and validation.

Voice cloning, large provider matrices, and advanced pronunciation scoring should wait until the end-to-end path is reliable.

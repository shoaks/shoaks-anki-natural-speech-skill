# Anki Natural Speech

A Codex workflow for building strict audio-first Anki decks.

The project is deliberately split by responsibility so card design, linguistic generation, TTS, and APKG mechanics cannot silently rewrite one another.

## Standard learner experience

**Front**

- Target audio replay control only
- no text, meaning, hint, transcription, or image

**Back**

1. Target
2. natural English meaning
3. Example 1 + its audio + tappable contextual word meanings
4. Example 2 + its audio + tappable contextual word meanings
5. reusable chunks and optional concise `More` information

A standard note therefore requires exactly **3 natural-audio artifacts**: Target + 2 examples.

## Skill architecture

```text
SKILL.md                         # orchestrator only
skills/
├── anki-card-format/
│   └── SKILL.md                # fixed UI / front-back contract
├── anki-content-generation/
│   └── SKILL.md                # Target, meaning, 2 examples, glosses, chunks
├── anki-audio-pipeline/
│   └── SKILL.md                # speech planning, TTS, audio validation/cache
└── anki-deck-builder/
    └── SKILL.md                # HTML/CSS/JS, media embedding, APKG verification
```

The root skill coordinates these four domains but does not duplicate their detailed rules.

## Why the split exists

The old monolithic skill mixed pedagogy, layout, language analysis, TTS selection, media handling, and packaging. That made implementation changes capable of changing learning semantics.

V3 enforces boundaries:

- card-format decides what the learner sees;
- content-generation decides what language content exists;
- audio-pipeline realizes fixed text as speech;
- deck-builder renders and packages already-validated data.

No downstream layer may weaken an upstream contract just to make a build pass.

## Machine-enforced contracts

- `schemas/note.schema.json` — exactly two examples and structured token/gloss data
- `schemas/speech-plan.schema.json` — provider-neutral speech plan
- `config.example.yaml` — provider, validation, and execution defaults
- `docs/codex-execution-contract.md` — fail-fast build discipline
- `docs/architecture.md` — responsibility boundaries

## TTS

No single TTS provider is mandatory.

Default behavior:

- prefer allowed local/free providers;
- do not silently select paid/network services;
- do not auto-download large models unless explicitly allowed;
- record the actual resolved provider/model;
- fail the build if any required Target or example audio cannot be generated.

## Interactive example glosses

Each lexical word in both examples is tappable/clickable.
The revealed meaning is contextual English, not a full dictionary dump.

When a multi-word chunk is the better learning unit, several word tokens may resolve to the same chunk gloss. JavaScript is progressive enhancement; the deck must retain a usable fallback.

## APKG completion rule

A deck is complete only after:

- all 3 required audio files per standard note exist and are non-empty;
- every required audio item has a valid `[sound:filename]` reference;
- all required audio files are physically embedded in APKG media;
- post-export inspection resolves every required sound reference;
- the card templates satisfy the fixed format contract.

Content JSON, TTS prompts, external audio paths, or a partially packaged APKG do not count as completion.

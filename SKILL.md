---
name: anki-natural-speech
summary: Orchestrate strict audio-first Anki card creation through separate format, content, audio, and deck-building skills.
---

# Anki Natural Speech — Orchestrator

## Purpose

This root skill is the workflow controller. It MUST NOT duplicate detailed card-design, linguistic-generation, TTS, or APKG rules.

The repository uses four authoritative sub-skills:

1. `skills/anki-card-format/SKILL.md`
   - learner-facing front/back layout
   - information hierarchy
   - tap-to-reveal gloss interaction
   - exactly two displayed examples

2. `skills/anki-content-generation/SKILL.md`
   - Target and NaturalMeaning
   - input correction policy
   - exactly two natural examples
   - contextual token/chunk glosses
   - reusable language chunks

3. `skills/anki-audio-pipeline/SKILL.md`
   - provider-neutral speech planning
   - TTS selection and synthesis
   - Target + Example 1 + Example 2 audio
   - audio validation and caching

4. `skills/anki-deck-builder/SKILL.md`
   - Anki field mapping
   - HTML/CSS/JS rendering
   - tappable gloss implementation and fallback
   - media embedding
   - APKG build and post-export verification

## Authority rule

Each sub-skill is authoritative only for its own domain.

A downstream skill MUST NOT alter an upstream skill's output semantics to make implementation easier.

Examples:

- the audio skill may not rewrite example text;
- the deck builder may not change two examples to four;
- the content skill may not redesign the card front;
- the card-format skill may not select a TTS provider.

If repository docs/config/schemas conflict with a sub-skill, update those artifacts before building.

## Mandatory execution order

Codex MUST execute this pipeline:

```text
User input
  -> read all four sub-skills
  -> content generation
  -> content/schema validation
  -> audio preflight
  -> synthesize Target + 2 Example audios
  -> audio validation
  -> card rendering
  -> APKG media embedding
  -> APKG export
  -> post-export validation
  -> completion report
```

## Preflight

Before batch work:

1. load all four sub-skills;
2. load `schemas/note.schema.json`;
3. load `config.example.yaml` or the active config;
4. check runtime and package-builder availability;
5. discover allowed TTS adapters without large automatic downloads;
6. verify writable cache/output paths;
7. run one representative note through the full vertical pipeline.

Do not start the batch until the representative note passes end-to-end.

## Default card invariant

The default study card is fixed:

- front: Target audio button only;
- back: Target, English NaturalMeaning, exactly two spoken examples, per-word contextual tap glosses, then optional `More`;
- exactly three required natural-audio files per standard note: Target + Example 1 + Example 2.

No other default structure is permitted unless the user explicitly changes the contract.

## Hard failure conditions

Do not export or report success if any standard note has:

- visible answer text or hints on the front;
- example count other than exactly 2;
- missing example English meaning;
- incomplete lexical gloss coverage;
- missing Target audio;
- missing Example 1 or Example 2 audio;
- zero-byte or unresolved audio;
- required sound files absent from APKG media;
- unresolved `[sound:filename]` references;
- a back layout that violates the card-format skill.

## TTS policy

No TTS engine is globally mandatory.
Use the audio skill's provider abstraction.
Prefer allowed local/free providers; do not silently use prohibited network or paid services.

## Input policy

Preserve the exact learner input in `OriginalInput`.
The content skill may create a corrected `Target` only when appropriate and must record the correction separately.

All learner-facing explanations are English only.

## Batch discipline

- validate one representative note first;
- cache successful media;
- do not retry unchanged deterministic failures;
- do not relax requirements to force a successful export;
- do not regenerate unchanged valid media unnecessarily.

## Completion language

Codex may say the deck is complete ONLY after the exported APKG passes post-export media and template validation.

A plan, JSON file, TTS prompt, filename, external audio path, or partially built APKG is not completion.

## Repository support files

- `schemas/note.schema.json` — machine-enforced note contract
- `schemas/speech-plan.schema.json` — provider-neutral speech-plan contract
- `config.example.yaml` — execution/provider/validation defaults
- `docs/architecture.md` — system boundaries and flow
- `docs/codex-execution-contract.md` — fail-fast execution discipline
- `examples/` — valid reference notes, never an alternate specification

If any support file contradicts the four sub-skills, the support file is stale and must be fixed before execution.

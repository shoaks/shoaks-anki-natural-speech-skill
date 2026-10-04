# TTS Provider Contract

The authoritative audio behavior is defined by `skills/anki-audio-pipeline/SKILL.md`.
This document defines provider-adapter mechanics only.

## Adapter interface

Each provider adapter should expose the equivalent of:

```python
class TTSProvider:
    id: str

    def probe(self) -> ProviderStatus:
        ...

    def supports(self, language: str, mode: str) -> bool:
        ...

    def synthesize(self, request: SynthesisRequest) -> AudioArtifact:
        ...
```

## ProviderStatus

Report:

- available / unavailable
- reason
- local vs network
- paid vs free
- supported languages when known
- model readiness
- whether a large download would be required

`probe()` must be cheap and must not download large models.

## SynthesisRequest

Provider-neutral data:

- text
- language
- dialect
- voice preference
- mode
- speech-plan guidance
- output path

Do not leak provider-specific command syntax into the core speech-plan schema.

## AudioArtifact

Return:

- actual provider id
- actual model id/name when known
- output file path
- stable media filename
- format
- duration when available
- validation status

The final note `AudioProvider` records the actual resolved provider id.

## Required synthesis set

For every standard pronounceable note, synthesize exactly the three required source texts supplied by the content layer:

1. Target
2. Example 1
3. Example 2

Each requires an independent playable artifact.
The provider layer must not invent, remove, merge, or rewrite these texts.

## Auto selection

With `provider: auto`:

1. enumerate registered adapters;
2. call cheap `probe()`;
3. exclude providers forbidden by local/network/paid policy;
4. exclude unsupported language/mode providers;
5. select according to configured preference;
6. synthesize;
7. if allowed, try another provider only after a bounded failure.

No provider name is globally mandatory.

## Failure categories

Treat these as deterministic unless evidence says otherwise:

- provider not installed
- unsupported language
- missing model with downloads disabled
- invalid credentials
- forbidden network/paid policy
- invalid configuration

Do not repeatedly retry deterministic failures.

## Packaging boundary

Provider success means only that valid local audio artifacts exist.
The `anki-deck-builder` skill owns `[sound:filename]` creation, APKG media embedding, and post-export resolution checks.

A standard note is incomplete unless all three validated audio artifacts are ultimately packaged and resolved inside the APKG.

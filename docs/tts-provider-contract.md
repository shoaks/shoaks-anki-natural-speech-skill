# TTS Provider Contract

The core skill is provider-neutral.

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

Should indicate:

- available / unavailable
- reason
- local vs network
- paid vs free
- supported languages when known
- model readiness
- whether a large download would be required

`probe()` should be cheap and must not download large models.

## SynthesisRequest

Provider-neutral data:

- text
- language
- dialect
- voice preference
- mode: natural / careful
- speech-plan guidance
- output path

Do not pass provider-specific command syntax through the core speech-plan schema.

## AudioArtifact

Return:

- actual provider id
- actual model id/name when known
- output file path
- format
- duration when available
- validation status

The final note `AudioProvider` must use the actual provider id returned by the adapter.

## Auto selection

With `provider: auto`:

1. enumerate registered adapters
2. call cheap `probe()`
3. remove providers forbidden by local/network/paid policy
4. remove providers that do not support the target language/mode
5. select according to configured preference
6. synthesize
7. if allowed, try another provider only after a bounded failure

No provider name is globally preferred by the skill specification.

## Failure categories

Treat these as deterministic unless evidence says otherwise:

- provider not installed
- unsupported language
- missing model with downloads disabled
- invalid credentials
- forbidden network/paid policy
- invalid configuration

Do not repeatedly retry deterministic failures.

Transient runtime failures may receive the configured single retry.

## Packaging invariant

Provider success is not enough.

The build succeeds only when:

- audio artifact exists
- file is non-zero
- note references it
- APKG media includes it
- post-export verification resolves the reference

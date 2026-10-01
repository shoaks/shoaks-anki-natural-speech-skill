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

The same request abstraction is used for the main Target and for each example sentence.

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

## Required synthesis set

For every normal pronounceable note, synthesize:

1. the Target;
2. Example 1;
3. Example 2;
4. Example 3;
5. Example 4;
6. any additional pronounceable examples.

Every example requires its own audio artifact. One combined recording does not replace the per-example files unless the note also retains individually addressable packaged audio for each example.

## Auto selection

With `provider: auto`:

1. enumerate registered adapters;
2. call cheap `probe()`;
3. remove providers forbidden by local/network/paid policy;
4. remove providers that do not support the target language/mode;
5. select according to configured preference;
6. synthesize;
7. if allowed, try another provider only after a bounded failure.

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

For every required Target/example audio artifact, the build succeeds only when:

- the artifact exists;
- the file is non-zero;
- the note/card references it through `[sound:filename]`;
- the file is physically included in APKG media;
- post-export verification resolves the reference.

An external path, URL, cache entry, or file left beside the APKG does not count as packaged audio.

If any required example audio fails this invariant, the whole note/build fails.

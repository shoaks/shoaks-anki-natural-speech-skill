---
name: anki-card-format
summary: Defines the immutable learner-facing layout and interaction contract for audio-first Anki cards.
---

# Anki Card Format Skill

## Authority

This skill owns ONLY learner-facing card structure, information hierarchy, and interaction behavior.
It does not choose TTS engines, synthesize audio, or package APKG files.
Other skills MUST NOT alter this layout.

## Default retrieval model

`sound -> direct comprehension -> reveal -> confirm -> transfer`

## Front — immutable

The default listening-card front contains ONLY the replayable Target audio control.

```text
┌──────────────────────────────┐
│                              │
│              🔊              │
│                              │
└──────────────────────────────┘
```

Forbidden on the front:

- Target text
- English meaning
- transcription
- hint
- image
- example
- grammar
- vocabulary
- pronunciation note
- tags
- any visible clue that reveals the answer

## Back — immutable order

```text
Target
NaturalMeaning

──────── Examples ────────

🔊 Example 1
[Example 1 sentence; every lexical word tappable]
Example 1 NaturalMeaning

🔊 Example 2
[Example 2 sentence; every lexical word tappable]
Example 2 NaturalMeaning

──────── More ────────
Reusable chunks / constructions
Optional concise grammar / usage / pronunciation / natural-speech note
```

Rules:

1. Exactly TWO core examples are shown.
2. Each example has its own replayable audio.
3. Each example has a natural English meaning.
4. Every lexical word in each example is tappable/clickable.
5. Tapping a word reveals its contextual English gloss.
6. When several words form the better learning unit, those words may resolve to the same chunk gloss.
7. Punctuation is not tappable and requires no gloss.
8. Glosses must not open an external dictionary by default.
9. `More` is always secondary and must appear after both examples.
10. Do not add extra visible sections unless the user explicitly changes this contract.

## Contextual gloss behavior

Preferred interaction:

`tap word -> show compact inline/popover contextual meaning -> tap another word to replace -> tap outside to close`

Example:

`Déjame pensar un momento.`

- `Déjame` -> `let me`; note: `deja + me; commonly used before an infinitive`
- `pensar` -> `to think`; note: `infinitive`
- `un` -> same chunk gloss as `momento`
- `momento` -> `a moment`; note: `un momento is a common conversational chunk`

This preserves per-word interaction while teaching chunks instead of fake one-token equivalence.

## No-JavaScript fallback

Interactive glosses are progressive enhancement.
The back must remain usable if JavaScript is unavailable.
Include the same gloss data in a compact secondary fallback that can be exposed without external navigation.

## Visual hierarchy

- Target is the strongest text element.
- NaturalMeaning is visually secondary.
- Examples are clearly separated but not card-within-card clutter.
- Gloss UI is hidden until requested.
- `More` is subdued and preferably collapsible.
- Light and dark mode must both remain readable.

## Completion gate

A default card is format-invalid if any of these are true:

- visible front content is not audio-only;
- back order differs from this skill;
- example count is not exactly 2;
- either example lacks audio or English meaning;
- any lexical example token lacks tappable gloss coverage;
- `More` appears before examples;
- interaction failure makes core learning content inaccessible.

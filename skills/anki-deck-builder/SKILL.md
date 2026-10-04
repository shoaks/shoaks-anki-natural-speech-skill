---
name: anki-deck-builder
summary: Render the fixed card template, embed media, build APKG files, and verify the exported deck.
---

# Anki Deck Builder Skill

## Scope

This skill owns rendering, Anki field mapping, HTML/CSS/JavaScript behavior, media embedding, APKG creation, and post-export verification.

It MUST NOT rewrite linguistic content, change the number of examples, choose a different card hierarchy, or invent audio.

## Inputs

Consume:

- validated structured note data from `anki-content-generation`;
- validated Target/Example audio artifacts from `anki-audio-pipeline`;
- layout/interaction rules from `anki-card-format`.

## Note fields

Canonical fields:

- `Target`
- `NaturalMeaning`
- `TargetAudio`
- `Example1Text`
- `Example1Meaning`
- `Example1Audio`
- `Example1TokensJSON`
- `Example1GlossesJSON`
- `Example2Text`
- `Example2Meaning`
- `Example2Audio`
- `Example2TokensJSON`
- `Example2GlossesJSON`
- `ChunksJSON`
- `MoreHTML`
- `OriginalInput`
- `CorrectionNote`
- metadata fields as required

Structured source data remains canonical; rendered HTML is derived output.

## Front template

The front MUST expose only the Target audio replay control.

Conceptually:

```html
<div class="front-audio">{{TargetAudio}}</div>
```

Do not render any other learner-visible field on the default front.

## Back template

Render in this exact sequence:

1. Target
2. NaturalMeaning
3. Examples heading/divider
4. Example 1 audio
5. Example 1 tappable sentence
6. Example 1 English meaning
7. Example 2 audio
8. Example 2 tappable sentence
9. Example 2 English meaning
10. More heading/divider
11. reusable chunks / optional secondary notes

## Tappable gloss rendering

Render every lexical token as an interactive span with a stable `GlossID` reference.

Example conceptual HTML:

```html
<span class="gloss-token" data-gloss-id="g2">pensar</span>
```

Punctuation is rendered as ordinary text.

Use lightweight JavaScript to:

- handle tap/click;
- find the corresponding gloss object;
- show one compact gloss popover/inline panel;
- replace the currently shown gloss when another word is selected;
- close on outside tap when practical.

Do not navigate away from the card and do not require an external dictionary.

## Fallback behavior

JavaScript is progressive enhancement.
Include a compact fallback rendering of all contextual glosses on the back so that the card remains usable if scripting is unavailable.

Fallback glosses must be visually secondary and must not clutter the default reading flow.

## Styling

Requirements:

- centered readable content width;
- strong Target hierarchy;
- secondary NaturalMeaning;
- clear Example separation;
- hidden gloss details until requested where scripting works;
- readable light and dark modes;
- adequate mobile tap targets;
- no unnecessary decorative UI.

## Media embedding

Every standard note has exactly three required audio artifacts:

- Target audio
- Example 1 audio
- Example 2 audio

For each:

1. verify the source file exists and is non-empty;
2. copy/include it in the Anki package media collection;
3. use a valid `[sound:filename]` reference;
4. ensure the note field points to the same filename;
5. after export, inspect the APKG media map and confirm the file is present.

External paths and URLs do not count as embedded media.

## APKG build

Preferred package builder: `genanki` or an equivalent library that produces valid Anki packages.

Use stable note/model/deck identifiers and stable media filenames so repeated builds are predictable.

## Validation gates

Before export, fail if:

- front contains any learner-visible content besides Target audio;
- example count is not exactly 2;
- either example lacks text, meaning, tokens, glosses, or audio;
- any lexical token lacks valid gloss resolution;
- any required audio file is missing or empty;
- any required `[sound:...]` reference is missing.

After export, fail if:

- APKG is unreadable;
- media map is empty for pronounceable notes;
- any required audio file is absent from package media;
- any required sound reference cannot be resolved;
- front/back template structure violates `anki-card-format`.

## Completion rule

Do not report deck generation as complete until post-export verification passes.

# Card Design V2 — Retired Reference

This document is no longer normative.

The authoritative card specification is:

`skills/anki-card-format/SKILL.md`

Current standard card contract:

- front: Target audio replay control only;
- no target text, meaning, image, hint, or transcription on front;
- back: Target -> NaturalMeaning -> Examples -> More;
- exactly 2 core examples;
- each example has its own audio and English meaning;
- every lexical example word is tappable for contextual English meaning;
- words belonging to a useful multi-word chunk may resolve to the same chunk gloss;
- JavaScript interaction is progressive enhancement with a no-JS fallback;
- `More` appears only after both examples.

Do not use historical V2 rules such as four-example minimums or optional images on the default front.

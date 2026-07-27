---
domain: design-system
paths:
  - "src/components/**"
  - "src/app/**/*.tsx"
  - "src/app/globals.css"
---

# design-system

Example pack. Replace every rule below with your own — a pack is only as good
as the rules it carries, and a copied one produces confident findings about
conventions you do not have.

Note there is no `adversary` or `operator` section. That is deliberate: a
section's presence is what puts a reviewer on the pitch, so omitting them
benches the security and ops reviewers on UI-only diffs instead of having them
hunt for injection vectors in a card component.

## standards

- Never hardcode a colour. Use the semantic tokens in `globals.css` — `text-error`, not `text-red-500`.
- Never hand-edit generated token files; change the source and re-sync.
- Icons go through the shared `<Icon>` wrapper, never a raw icon-library import.
- Server-by-default. `"use client"` only for state, effects, handlers, refs, browser APIs.
- A hand-rolled button, dialog or input that duplicates a `ui/` primitive is a finding — cite the primitive.

## spec

- Renders at the stated minimum viewport with no horizontal scroll.
- Empty, loading and error states exist for anything that fetches.
- Interactive states the design shows are implemented: hover, focus-visible, disabled, pressed.
- Copy matches the design's copy exactly, including capitalisation.

## prover

- A component with conditional rendering has a test per branch — especially empty and error states.
- If the animation library is mocked in the test setup, a test asserting on animated presence may be asserting on the mock. Mutate the conditional and check it still fails.

## steward

- Public pages only. Keyboard-operable with a visible focus ring, labelled inputs, alt text, accessible names on icon-only controls, ordered headings, no meaning by colour alone.
- Check contrast against the token pair actually rendered, not the palette in the abstract.

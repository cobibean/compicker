---
name: compicker
description: Build temporary in-product pickers for comparing visual or UI alternatives, then help choose and lock the winning option. Use when the user says /compicker, $compicker, compicker, compare/pick visual variants, make a review picker, compare in a design-lab route/page, compare backgrounds, compare hero images, compare component designs, compare copy/layout/card treatments, or wants to audition multiple UI directions inside the actual app before committing.
---

# Compicker

## Overview

Use this skill to compare visual/UI alternatives in the product itself. The default output is a temporary picker that makes variants easy to toggle, plus a clean path to remove the picker and lock the chosen direction. Use an inline picker for narrow surfaces; use a temporary design-lab route/page when variants need broader page context.

## Workflow

1. Identify what is being compared: backgrounds, components, copy treatments, layouts, colors, media, or interaction states.
2. Find the current implementation and variant sources. Read local code/assets first, then ask only if variants or target surfaces are genuinely ambiguous.
3. Choose the comparison surface. Prefer a temporary control near the affected surface for narrow changes. Create a temporary design-lab route/page when the user asks for one, the comparison spans multiple contexts, or an inline picker would obscure or distort the thing being judged.
4. Wire variants to the exact layer being compared. Be explicit about main background versus card background, section background versus component surface, copy text versus layout, and hover/active states.
5. Verify each option by clicking through it in-browser. Check desktop and mobile when the visual surface is responsive.
6. Summarize the strongest option and tradeoffs. When the user chooses, lock the winner and remove the temporary picker code.

## Design Lab Route Pattern

Use a `/design-lab` route, or the project's nearest equivalent, when comparison needs a focused review surface with real page context. Do not create `/design-lab` automatically for every compicker task.

Create a design lab when:

- The user explicitly asks for `/design-lab`, a design lab, a review page, or a sandbox route.
- The variants affect page-level systems such as theme, navigation, hero treatment, typography, motion language, card systems, modal chrome, or shared tokens.
- The variants need to be seen in several realistic contexts at once, such as hero plus nav plus cards plus modal.
- Placing the picker inline would crowd, obscure, or change the surface being judged.

Keep the lab useful and temporary:

- Reuse real app assets, copy, components, sections, and interaction states whenever possible.
- Show variants in context, not as isolated specimens on a blank page.
- Include the temporary picker, current selection, and at least one realistic target surface using that selection.
- Scope lab-only state with stable `data-*` attributes, route-local state, or CSS variables. Do not wire global behavior permanently until the user chooses a winner.
- Mark the lab as temporary and provide a path back to the real app.
- When the user chooses, either remove the lab route with the picker or promote only the winning durable pieces into the app.

## Picker Pattern

Use a robust picker rather than clever CSS when the user will click it repeatedly:

- For React/Next apps, prefer a tiny client component with `useState`, `aria-pressed`, and a stable `data-*` attribute or CSS variable on the relevant parent.
- For static HTML/CSS prototypes, radios plus `:checked`/`:has()` are acceptable only if the browser target supports them and the selected state is visibly reliable.
- Keep the picker visibly temporary. Do not make it look like a product feature unless the user asks for a permanent variant selector.
- Use short human labels, not filenames, unless filenames are the clearest labels.
- Cap visible options around 3 to 7. If there are more, group them or create a two-step picker.

Example React shape:

```tsx
"use client";

import { useEffect, useState } from "react";

export function Compicker({ options }: { options: Array<{ id: string; label: string }> }) {
  const [selected, setSelected] = useState(options[0]?.id ?? "");

  useEffect(() => {
    document.querySelector<HTMLElement>("[data-compicker-scope]")?.setAttribute("data-compicker", selected);
  }, [selected]);

  return options.map((option) => (
    <button
      aria-pressed={selected === option.id}
      key={option.id}
      onClick={() => setSelected(option.id)}
      type="button"
    >
      {option.label}
    </button>
  ));
}
```

## Visual Rules

- Compare in context, not in isolation. The chosen variant must work with adjacent text, controls, cards, navigation, and responsive constraints.
- Preserve the existing product direction unless the user explicitly wants a new direction.
- Avoid picker-induced layout shift. Give controls stable dimensions and place them where they do not obscure the thing being judged.
- If comparing images or backgrounds, always name the layer: page background, hero panel, card texture, modal backdrop, object preview, etc.
- Keep readability overlays separate from the variant itself so the user can judge composition and contrast fairly.
- Do not leave dead picker CSS, temporary client components, review labels, test query params, or unused assets after locking the winner.

## Verification

Before reporting done:

- Run the relevant typecheck/lint/build command when code changed.
- Open the app in the browser when possible.
- Click every picker option and confirm the intended layer changes.
- Capture or inspect desktop and mobile views for visual surfaces that can wrap, crop, overlap, or hide important content.
- If browser tooling is blocked, verify via server HTML, DOM state, computed selectors, and source inspection, and say what could not be visually confirmed.

## Lock-In Pass

When the user chooses a winner:

1. Replace dynamic picker state with the chosen static value.
2. Remove picker markup, component files, styles, imports, and references.
3. Restore spacing that existed only to fit the picker.
4. Keep any useful permanent variables, tokens, or asset choices.
5. Re-run checks and mention any remaining warnings that were pre-existing.

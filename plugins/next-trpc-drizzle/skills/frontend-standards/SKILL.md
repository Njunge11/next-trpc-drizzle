---
name: frontend-standards
description: Use when building or reviewing UI, including when given a UI to build from a description — shadcn/ui-first, semantic design tokens (no hardcoded colors, px, or inline styles), and mobile-first responsive that explicitly handles tablet (md breakpoint).
---

# Frontend Standards

How UI gets built in `apps/admin`. Stack: **shadcn/ui + Tailwind CSS**. Three rules, because these are where agent-built UI usually goes wrong: hardcoded values, reinvented components, and broken tablet layouts.

## 1. shadcn first — don't reinvent

- Build from **shadcn/ui primitives** (Button, Card, Dialog, Table, Tabs, Form, Input, Badge, …) and their built-in `variant`/`size` props. Compose them; don't hand-roll something shadcn already provides.
- Add a primitive with `npx shadcn@latest add <component> --overwrite`. **Never hand-write or hand-edit** a primitive's source.
- Build a custom component **only when explicitly instructed**. Otherwise stick to what shadcn offers.

## 2. No hardcoded values — use tokens and the scale

- **Colors → semantic tokens only.** Use the shadcn CSS-variable tokens, never hex, never inline-style colors, and not raw palette shades (e.g. `bg-stone-500`) unless instructed:
  - surfaces/text: `bg-background` / `text-foreground`, `bg-card` / `text-card-foreground`, `bg-popover` / `text-popover-foreground`
  - actions: `bg-primary` / `text-primary-foreground`, `bg-secondary` / `text-secondary-foreground`, `bg-destructive`
  - subtle/interactive: `bg-muted` / `text-muted-foreground`, `bg-accent` / `text-accent-foreground`
  - lines/focus: `border-border`, `border-input`, `ring-ring`; charts: `chart-1`…`chart-5`
  - Pairing rule: the base token sets the **surface**, the `-foreground` token sets **text/icon** on it.
- **Spacing, sizing, typography → Tailwind scale utilities** (`p-4`, `gap-2`, `h-10`, `text-sm`). **No arbitrary values** (`w-[327px]`, `mt-[13px]`) and **no inline `style={{}}`** unless explicitly instructed.

## 3. Responsive — mobile-first, every breakpoint, never skip tablet

Tailwind breakpoints (min-width, and they apply **at that width and up**):

| prefix | width | use |
| --- | --- | --- |
| _(none)_ | 0 | mobile baseline |
| `sm` | 640px | large phone |
| `md` | **768px** | **tablet** |
| `lg` | 1024px | desktop |
| `xl` | 1280px | wide |
| `2xl` | 1536px | extra-wide |

- **Mobile-first.** Unprefixed utilities are the mobile baseline; layer `sm:`/`md:`/`lg:` to override upward. Never use `sm:` to mean "on mobile" — `sm:` is "at the small breakpoint **and up**."
- **The tablet trap (the #1 bug here):** don't jump from base straight to `lg:` and leave `md:` (768–1023px) unstyled — that's why tablet looks broken. Every responsive layout must have a deliberate `md:` state.
- **Fluid, not fixed.** Use responsive grid/flex (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3`, `flex-col md:flex-row`); avoid fixed pixel widths that overflow on tablet.
- **Verify at three widths before done:** ~375px (mobile), **~768px (tablet)**, ~1280px (desktop).

## Building from a description

You'll usually get the UI as prose. Then:

- Map each described element to the **closest shadcn primitive**; theme it with tokens; implement all breakpoints (incl. `md:`).
- **Preserve copy verbatim.** Don't invent labels, headings, overlines, or placeholder text that weren't described.
- Match existing patterns in the app. Ask only when genuinely ambiguous; otherwise proceed.

## Sources

- [shadcn/ui — Theming (semantic tokens)](https://ui.shadcn.com/docs/theming)
- [Tailwind CSS — Responsive design (breakpoints, mobile-first)](https://tailwindcss.com/docs/responsive-design)

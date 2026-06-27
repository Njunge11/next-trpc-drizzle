---
name: frontend-standards
description: Use when building or reviewing UI — composing features from shadcn/ui primitives, semantic design tokens (no hardcoded colors/px/inline styles), and mobile-first responsive that handles tablet. Includes building from a described UI.
---

# Frontend Standards

How UI gets built. Stack: **shadcn/ui (+ Radix) + Tailwind CSS**. We **compose existing shadcn components** into features — we rarely build components from scratch. shadcn components already embody the [components.build](https://components.build) standard (accessible, compound, variant-driven, token-themed), so the job is assembling them well, not re-authoring them.

## Compose shadcn — the default

- **Don't reinvent.** Use shadcn primitives (`Button`, `Card`, `Dialog`, `Table`, `Tabs`, `Form`, `Input`, `Select`, `Badge`, …). Need one that isn't installed? `npx shadcn@latest add <component>`. **Never hand-edit a primitive's source.** Build something custom **only when explicitly instructed** (see [Authoring](#authoring-a-custom-component-rare)).
- **Decompose the UI into a tree of shadcn parts** by responsibility and repetition — not one monolith. A described "card list" → `Card` + `CardHeader`/`CardTitle`/`CardContent`, a row component, etc.
- **Use the component's own API:** its `variant`/`size` props for permutations, and its **compound parts** as intended (`Dialog.Root`/`Trigger`/`Content`, `Card` + `CardHeader`/`CardTitle`/`CardFooter`). Don't recreate what a prop already does.
- **Compose with `children`/slots**, not data/boolean prop piles.
- **`asChild` to integrate** — render a shadcn part's behavior on your element with no wrapper: a `Button` that's actually a Next `<Link>` (`<Button asChild><Link href>…</Link></Button>`), `DropdownMenu.Trigger asChild` wrapping your `Button`. The child must be a **single, non-Fragment** element that spreads props.
- **Preserve copy verbatim** — don't invent labels, headings, or placeholder text that weren't described. Match existing patterns; ask only when genuinely ambiguous.

## No hardcoded values — tokens + scale

- **Colors → semantic tokens only**, never hex/inline-color/raw palette (`bg-stone-500`) unless instructed: `bg-background`/`text-foreground`, `bg-card`/`text-card-foreground`, `bg-primary`/`text-primary-foreground`, `bg-secondary`/`…`, `bg-muted`/`text-muted-foreground`, `bg-accent`/`…`, `bg-destructive`, `border-border`/`border-input`, `ring-ring`, `chart-1…5`. Base token = surface, `-foreground` = text/icon on it.
- **Spacing/sizing/typography → Tailwind scale** (`p-4`, `gap-2`, `h-10`, `text-sm`). **No arbitrary values** (`w-[327px]`, `mt-[13px]`) and **no inline `style={{}}`** unless instructed. Merge classes with `cn(...)`; your `className` goes last so it can override.

## Responsive — mobile-first, never skip tablet

Tailwind breakpoints apply **at that width and up**: `sm` 640 · **`md` 768 (tablet)** · `lg` 1024 · `xl` 1280 · `2xl` 1536.

- **Mobile-first:** unprefixed = mobile baseline; layer `sm:`/`md:`/`lg:` upward. `sm:` is "at the small breakpoint and up," never "on mobile."
- **The tablet trap (#1 bug):** don't jump base→`lg:` and leave `md:` (768–1023) unstyled. Every responsive layout needs a deliberate `md:` state.
- **Fluid, not fixed:** `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`, `flex-col md:flex-row`; avoid fixed widths that overflow tablet. Verify at ~375 / **~768** / ~1280.

## Accessibility — don't undo what shadcn gives you

shadcn/Radix ship keyboard nav, focus management, and ARIA for free. The consumer's job is to **not break it**:

- Keep **semantic elements** and use the right primitive (a `Dialog`, not a styled `div`). Don't strip `aria-*`/roles off shadcn parts.
- **Icon-only buttons need an accessible name** — `aria-label` (+ icon `aria-hidden`). **Use `<label>`s, not placeholders.** Don't convey meaning by color alone. Keep the `:focus-visible` ring.

## Authoring a custom component (rare)

Only when shadcn genuinely lacks it **and** you're told to build custom. Then follow the [components.build](https://components.build) standard:

- **Compound pattern** — a `Root` holds state in Context; named parts (`Trigger`/`Content`/`Item`/`Header`/`Title`/…) read it. **One component = one element.**
- **Extend native props** (`React.ComponentProps<"div">`), **spread props last**, export `<Name>Props`, avoid HTML-colliding prop names (`title`).
- **`asChild` via Radix `Slot`** (`asChild ? Slot : "button"`); **controlled + uncontrolled** via `useControllableState`; **`data-state`** for styleable state and **`data-slot`** for parent targeting.
- **Accessibility is yours to implement:** semantic HTML, keyboard map, ARIA roles/states, focus management. See the standard's pages below.

## Sources

- [components.build](https://components.build) — the standard (co-authored by shadcn): [composition](https://components.build/composition) · [types](https://components.build/types) · [as-child](https://components.build/as-child) · [polymorphism](https://components.build/polymorphism) · [state](https://components.build/state) · [styling](https://components.build/styling) · [design-tokens](https://components.build/design-tokens) · [data-attributes](https://components.build/data-attributes) · [accessibility](https://components.build/accessibility)
- [shadcn/ui — Theming](https://ui.shadcn.com/docs/theming) · [Tailwind — Responsive design](https://tailwindcss.com/docs/responsive-design)

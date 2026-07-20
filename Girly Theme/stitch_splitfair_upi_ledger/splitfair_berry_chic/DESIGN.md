---
name: SplitFair Berry Chic
colors:
  surface: '#151215'
  surface-dim: '#151215'
  surface-bright: '#3c383b'
  surface-container-lowest: '#100d10'
  surface-container-low: '#1e1b1d'
  surface-container: '#221f21'
  surface-container-high: '#2d292c'
  surface-container-highest: '#383436'
  on-surface: '#e8e0e4'
  on-surface-variant: '#cfc3cc'
  inverse-surface: '#e8e0e4'
  inverse-on-surface: '#332f32'
  outline: '#988e96'
  outline-variant: '#4d444c'
  surface-tint: '#e4b9e3'
  primary: '#e4b9e3'
  on-primary: '#442547'
  primary-container: '#4a2b4d'
  on-primary-container: '#ba92bb'
  inverse-primary: '#755377'
  secondary: '#c9c6be'
  on-secondary: '#31302b'
  secondary-container: '#484740'
  on-secondary-container: '#b8b5ad'
  tertiary: '#ffb2bf'
  on-tertiary: '#660028'
  tertiary-container: '#71002d'
  on-tertiary-container: '#ff6f92'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#ffd6fe'
  primary-fixed-dim: '#e4b9e3'
  on-primary-fixed: '#2c1030'
  on-primary-fixed-variant: '#5c3b5e'
  secondary-fixed: '#e6e2d9'
  secondary-fixed-dim: '#c9c6be'
  on-secondary-fixed: '#1c1c16'
  on-secondary-fixed-variant: '#484740'
  tertiary-fixed: '#ffd9de'
  tertiary-fixed-dim: '#ffb2bf'
  on-tertiary-fixed: '#3f0016'
  on-tertiary-fixed-variant: '#90003b'
  background: '#151215'
  on-background: '#e8e0e4'
  surface-variant: '#383436'
typography:
  headline-lg:
    fontFamily: Fraunces
    fontSize: 40px
    fontWeight: '700'
    lineHeight: 48px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Fraunces
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 38px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Fraunces
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-sm:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  data-lg:
    fontFamily: IBM Plex Mono
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  data-md:
    fontFamily: IBM Plex Mono
    fontSize: 14px
    fontWeight: '500'
    lineHeight: 20px
  label-caps:
    fontFamily: IBM Plex Mono
    fontSize: 12px
    fontWeight: '700'
    lineHeight: 16px
    letterSpacing: 0.1em
spacing:
  unit: 4px
  gutter: 16px
  margin-mobile: 20px
  margin-desktop: 64px
  receipt-width: 360px
---

## Brand & Style
This design system variant reimagines the utility of expense splitting through a chic, feminine lens. The brand personality is playful and sophisticated, balancing the precision of a financial tool with the aesthetic of a high-end boutique or editorial magazine. 

The style is a blend of **Tactile Minimalism** and **Neo-Editorial**. It leverages the physical metaphor of a receipt—specifically the "receipt-strip" aesthetic—incorporating perforated edges and monospaced data, but softens the experience with a lush, berry-toned palette. The emotional response should be one of "effortless organization": making the often-dry task of settling debts feel like a curated, social ritual.

## Colors
The palette shifts the foundation from corporate navy to a deep, soulful plum. 

- **Primary (#4A2B4D):** Used for the main application background and deep-set containers. It provides a luxurious, high-contrast base for the paper elements.
- **Secondary (#F7F3EA):** The "Warm Paper" surface. This is used for all foreground cards, receipt strips, and primary content areas to maintain a tactile, physical feel.
- **Tertiary (#D81B60):** The "Vibrant Rose." This is the semantic color for "owed" states, debts, and urgent actions. It is high-energy and authoritative.
- **Accent (#F48FB1):** The "Soft Blush." Used for interactive elements, primary CTAs, and decorative flourishes. It provides a friendly, approachable contrast to the deep plum background.

## Typography
The typography is a critical pillar of this design system's identity. 

- **Fraunces** is used for headlines to provide a soft, editorial, and "literary" feel. Its unique soft-serif terminals complement the feminine aesthetic.
- **IBM Plex Mono** is used exclusively for financial data, timestamps, and receipt-style labels. This maintains the "SplitFair" heritage of data-driven precision within a soft environment.
- **Inter** handles all standard body copy to ensure maximum legibility and a clean, modern interface.

## Layout & Spacing
The layout follows a **Hybrid Fluid-Fixed** model. While the overall interface is responsive, the core transactional content is housed within "Receipt Strips"—fixed-width containers (max 360px) that float or stack within the layout.

- **The Receipt Strip:** Centrally aligned on desktop, full-width with 20px margins on mobile.
- **Perforated Edges:** Every card/receipt must feature a 4px tall jagged or "zig-zag" mask at the top and bottom.
- **Rhythm:** Uses an 8px base grid. Padding inside cards is a generous 24px to evoke a premium feel.

## Elevation & Depth
Depth is created through **Tonal Stacking** rather than traditional shadows. 

1. **Floor:** The Deep Berry (#4A2B4D) background.
2. **Elevated Surface:** The Warm Paper (#F7F3EA) card. This uses a very subtle, low-opacity Plum shadow (10% opacity, 20px blur) to appear as if it is resting on the berry surface.
3. **Interactive Elements:** Buttons and chips use high-contrast fills (Blush or Rose) to pop against the paper, rather than shadow-based lift.

## Shapes
In line with the "Receipt" metaphor, the design system utilizes **Sharp (0px)** corners for all primary containers and card elements. The "softness" of the brand is communicated through color and typography rather than rounded corners.

Circular elements are reserved exclusively for:
- User Avatars.
- Radio button indicators.
- Decorative "hole-punch" icons at the side of cards.

## Components

- **The Receipt Card:** A sharp-edged Warm Paper (#F7F3EA) container. It features a dashed line separator (`border-style: dashed`) to divide line items. The top and bottom edges utilize the signature zig-zag perforation.
- **Primary Button:** Solid Blush (#F48FB1) background with Deep Berry (#4A2B4D) text. Sharp corners. Bold, monospaced labels.
- **Status Chips:** Small, monospaced tags. "Owed" uses a Rose (#D81B60) outline; "Paid" uses a subtle Plum outline.
- **Input Fields:** Bottom-border only (1px solid Plum) to mimic a physical form. The placeholder text uses the softest shade of the primary plum.
- **Amount Display:** Always rendered in IBM Plex Mono. Negative balances or debts are highlighted in Vibrant Rose (#D81B60).
- **The "Tear" Interaction:** When a debt is settled, the UI uses a horizontal "tear" animation, visually splitting the receipt component.
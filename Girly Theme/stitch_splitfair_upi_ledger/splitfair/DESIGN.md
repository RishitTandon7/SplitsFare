---
name: SplitFair
colors:
  surface: '#131314'
  surface-dim: '#131314'
  surface-bright: '#3a3939'
  surface-container-lowest: '#0e0e0e'
  surface-container-low: '#1c1b1c'
  surface-container: '#201f20'
  surface-container-high: '#2a2a2a'
  surface-container-highest: '#353535'
  on-surface: '#e5e2e2'
  on-surface-variant: '#c5c6ca'
  inverse-surface: '#e5e2e2'
  inverse-on-surface: '#313030'
  outline: '#8f9194'
  outline-variant: '#44474a'
  surface-tint: '#c3c7cc'
  primary: '#c3c7cc'
  on-primary: '#2d3135'
  primary-container: '#14181c'
  on-primary-container: '#7e8186'
  inverse-primary: '#5b5f63'
  secondary: '#c9c6be'
  on-secondary: '#31302b'
  secondary-container: '#484740'
  on-secondary-container: '#b8b5ad'
  tertiary: '#e8c17b'
  on-tertiary: '#412d00'
  tertiary-container: '#221500'
  on-tertiary-container: '#9d7c3d'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#e0e3e8'
  primary-fixed-dim: '#c3c7cc'
  on-primary-fixed: '#181c20'
  on-primary-fixed-variant: '#43474c'
  secondary-fixed: '#e6e2d9'
  secondary-fixed-dim: '#c9c6be'
  on-secondary-fixed: '#1c1c16'
  on-secondary-fixed-variant: '#484740'
  tertiary-fixed: '#ffdea7'
  tertiary-fixed-dim: '#e8c17b'
  on-tertiary-fixed: '#271900'
  on-tertiary-fixed-variant: '#5d4207'
  background: '#131314'
  on-background: '#e5e2e2'
  surface-variant: '#353535'
typography:
  display-lg:
    fontFamily: Fraunces
    fontSize: 40px
    fontWeight: '700'
    lineHeight: 48px
  display-lg-mobile:
    fontFamily: Fraunces
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
  headline-md:
    fontFamily: Fraunces
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-lg:
    fontFamily: Inter
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-mono:
    fontFamily: IBM Plex Mono
    fontSize: 14px
    fontWeight: '500'
    lineHeight: 20px
    letterSpacing: 0.02em
  amount-lg:
    fontFamily: IBM Plex Mono
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  amount-sm:
    fontFamily: IBM Plex Mono
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  unit: 4px
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 32px
  receipt-padding: 20px
---

## Brand & Style

The design system is built on the metaphor of the "Digital Ledger." It bridges the gap between traditional Indian bookkeeping (Bahi Khata) and modern UPI-led digital payments. The aesthetic is **Tactile & Precision-focused**, combining the warmth of physical paper with the sharp utility of a financial tool.

The brand personality is reliable yet approachable, evoking the feeling of a trusted local merchant’s ledger. It utilizes a **Tactile-Minimalist** style: clean layouts punctuated by physical metaphors like perforated edges, dashed lines, and high-contrast ink-on-paper typography. The emotional goal is to make debt feel less stressful and more like a shared, organized record of trust.

## Colors

The palette is anchored in **Ink Navy**, serving as a deep, authoritative background that minimizes eye strain during late-night bill splits. 

- **Surface Strategy:** The "Receipt Strip" uses **Warm Paper** (#F7F3EA) as a high-contrast surface against the dark background, creating an immediate focal point for transaction data.
- **Functional Color:** **Deep Jade** and **Burnt Rust** are used exclusively for financial status—owing and being owed. They are tuned for legibility against both the dark canvas and the light receipt surfaces.
- **Accents:** **Soft Gold** is reserved for primary actions (CTA) and UPI-related elements, subtly referencing the metallic foils found on Indian currency.

## Typography

This design system employs a three-tier typographic hierarchy to ensure clarity and character:

1.  **Display (Fraunces):** Used for large balances, screen titles, and significant headers. Its high-contrast serifs provide a literary, authoritative feel.
2.  **Body (Inter):** Used for all UI copy, descriptions, and buttons. It provides a clean, neutral balance to the more expressive display face.
3.  **Numeric/Mono (IBM Plex Mono):** Strictly for all currency amounts and dates. Tabular figures ensure that numbers align perfectly in lists, mimicking the precision of a printed receipt.

*Note: Always use the Currency Symbol (₹) in the Mono font to maintain alignment.*

## Layout & Spacing

The layout follows a **Fluid Grid** model optimized for one-handed mobile use. 

- **The Ledger Column:** Content is primarily centered in a single column with 16px lateral margins.
- **Vertical Rhythm:** A strict 4px baseline grid ensures that monospaced numbers and body text align across components.
- **Visual Breaks:** Use dashed horizontal rules (1px, 4px dash, 4px gap) instead of solid lines to separate receipt items, reinforcing the paper metaphor.
- **Safe Areas:** Bottom-fixed CTA buttons must account for mobile home-bar "safe areas" with a minimum of 24px bottom padding.

## Elevation & Depth

Depth is conveyed through **Tonal Stacking** and physical motifs rather than traditional drop shadows.

- **Level 0 (Background):** Ink Navy (#14181C).
- **Level 1 (Receipt Strips):** Warm Paper (#F7F3EA). These do not use shadows; instead, they rely on the high contrast against the dark background to "pop."
- **Level 2 (Active States):** Soft Gold (#D8B26E) overlays or subtle inner shadows for pressed buttons.
- **Physical Depth:** Use a "perforated" mask on the bottom edge of cards. This "zigzag" or "torn paper" edge serves as the primary indicator of an expandable or scrollable element.

## Shapes

The shape language is **Soft (0.25rem)** to maintain the feel of cut paper and ledger books. 

- **Buttons & Inputs:** Use the standard 4px (Soft) radius.
- **Receipt Strips:** The top corners are 4px, while the bottom edge is a custom "perforated" or "torn" vector path. 
- **Checkboxes:** Keep sharp or minimally rounded (2px) to mimic the "box" in a physical ledger.

## Components

### Receipt Strip Cards
The primary container for transaction details. Use **Warm Paper** background with **Muted Ink-Grey** text. The bottom must feature a jagged/torn edge. Use dashed dividers for internal sections.

### Buttons
- **Primary CTA:** Soft Gold background, Ink Navy text, Bold Inter.
- **UPI Quick-Pay:** Solid Deep Jade with a white/light-paper icon for "Pay Now."
- **Secondary:** Outlined with 1px Muted Ink-Grey, no fill.

### Settlement Panels
A specialized ledger summary. Use a two-column layout: Left (Description in Inter), Right (Amount in IBM Plex Mono). Highlight the "Net Balance" with a background tint of either Jade or Rust depending on the sign.

### Input Fields
Minimalist underline-style inputs (mimicking a line in a notebook). When focused, the underline transitions from Ink-Grey to Soft Gold. Use IBM Plex Mono for all numeric input fields.

### Chips & Tags
Small, rectangular containers with 2px radius. Use for categories (e.g., "Food", "Rent"). Use low-opacity tints of the Neutral Ink color.
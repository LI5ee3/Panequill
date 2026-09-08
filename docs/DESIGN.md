# Panequill Design

## 1. Purpose

This document defines the authoritative design rules for Panequill.

Panequill is a personal web dashboard intended primarily for use as a browser start page. Its interface should remain visually calm, immediately readable, and suitable for frequent long-term use.

This document defines Panequill-specific design decisions and establishes how Material Design 3 is applied within the product.

Implementation details belong in `TECH_STACK.md` and the codebase. Product scope and behavior belong in `PRODUCT.md`.

---

## 2. Design Foundation

Panequill uses **Material Design 3 (MD3)** as its sole foundational design system.

Material Design 3 is the default authority for standard interface behavior, including:

- color roles
- typography
- shape
- elevation
- motion
- interaction states
- accessibility
- responsive and adaptive behavior
- standard UI components

Panequill does not define alternative conventions where Material Design 3 already provides an appropriate solution.

### 2.1 M3 Expressive

**M3 Expressive is explicitly out of scope.**

Panequill must not adopt M3 Expressive-specific visual or interaction patterns unless this document is explicitly revised in the future.

This includes avoiding the introduction of M3 Expressive conventions merely because they appear in newer Material documentation or tooling.

### 2.2 Design Precedence

Design decisions follow this precedence order:

1. `DESIGN.md`
2. Material Design 3
3. existing established Panequill implementation patterns

If a significant design decision is not covered by any of the above, it must not be invented implicitly during implementation.

Panequill-specific rules may extend or override Material Design 3 only where required by the product.

---

## 3. Design Principles

### 3.1 Content First

The dashboard exists to present useful personal information.

Visual treatment must support the content rather than compete with it.

Decorative styling must never reduce readability, information hierarchy, or usability.

### 3.2 Calm by Default

Panequill is intended to remain visible as a browser start page and should therefore avoid unnecessary visual activity.

The resting interface should contain only information and controls that are useful in the current context.

Editing controls, configuration actions, and secondary actions should remain unobtrusive until required.

### 3.3 Consistency Before Novelty

Widgets and controls must appear to belong to one coherent product.

Individual widgets must not introduce independent design languages, arbitrary shape systems, unrelated color palettes, or inconsistent interaction behavior.

Existing Panequill and Material Design 3 patterns should be reused before introducing new patterns.

### 3.4 Wallpaper-Aware

The Bing Daily Wallpaper is a permanent part of the Panequill visual environment.

All dashboard surfaces must remain readable across backgrounds with substantially different brightness, contrast, detail, and color.

Wallpaper visibility is important, but information readability always takes precedence.

### 3.5 Progressive Disclosure

Panequill should expose complexity only when it is needed.

The default dashboard state prioritizes consumption.

Editing, configuration, removal, resizing, and similar management actions should appear contextually rather than remain permanently visible.

---

## 4. Wallpaper

Panequill uses Microsoft Bing Daily Wallpaper as the dashboard background.

The wallpaper is not a decorative optional theme. It is part of the core presentation of the product.

### 4.1 Presentation

The wallpaper should:

- occupy the full dashboard viewport
- preserve its original aspect ratio
- fill the available viewport without distortion
- use cropping when necessary rather than stretching
- remain visually prominent behind dashboard content

The exact crop may vary with viewport dimensions.

### 4.2 Readability

Widget readability must not depend on the wallpaper having a particular brightness or color.

Surface treatment may use techniques such as:

- translucent surfaces
- backdrop blur
- subtle outlines
- controlled overlays
- tonal separation

These techniques exist to maintain readability and visual separation.

They must not become decorative effects in their own right.

### 4.3 Wallpaper Obstruction

Large opaque regions should not be introduced without a functional reason.

The dashboard should preserve a meaningful visual relationship with the wallpaper while maintaining sufficient content contrast.

---

## 5. Color and Surfaces

Panequill follows the Material Design 3 semantic color-role model.

Color should represent role and state rather than individual widget identity.

Standard Material roles should be used wherever applicable, including concepts equivalent to:

- primary
- on-primary
- surface
- on-surface
- surface variants
- outline
- error
- on-error

Panequill may define additional semantic roles only where Material Design 3 does not represent a product-specific requirement.

Examples may include:

- wallpaper scrim
- widget surface
- widget editing surface

Additional roles must remain semantic and reusable.

### 5.1 Widget Color

Widgets should use a predominantly neutral surface language.

A weather widget, bookmark widget, clock widget, or other content type should not receive a unique base color merely to distinguish its category.

Color may be used when it communicates meaningful information or state.

### 5.2 Accent Color

Accent color should be used selectively.

It should primarily indicate:

- active interaction
- selection
- focus
- important actions
- semantic emphasis

Accent color should not become general decoration.

### 5.3 Transparency

Transparency must never be prioritized over readability.

A widget surface must remain usable over both high-key and low-key wallpapers.

---

## 6. Typography

Panequill follows Material Design 3 typography principles.

Typography should establish a clear hierarchy while keeping the number of distinct visual levels limited.

Panequill primarily requires the following functional hierarchy:

- prominent display information
- widget title
- primary content
- secondary content
- supporting metadata

Examples of prominent display information include clocks, dates, temperatures, or other values whose size is part of their functional hierarchy.

Typography must remain consistent across widgets.

Widget implementations must not create arbitrary independent font scales.

The final typography token mapping should use Material Design 3 roles appropriate to each use case.

---

## 7. Dashboard Layout

The dashboard is the primary spatial environment of Panequill.

### 7.1 Grid

Widgets should be positioned using a consistent dashboard grid.

Widget placement should be based on grid units rather than arbitrary pixel coordinates.

The grid should provide:

- consistent alignment
- consistent gaps
- predictable widget sizing
- stable drag behavior
- stable resize behavior

Exact grid dimensions are implementation decisions and should be established when validated against the actual dashboard interface.

### 7.2 Spatial Balance

The dashboard should remain visually balanced with both sparse and dense widget arrangements.

Artificial content should not be added merely to fill empty space.

Whitespace is valid and should not automatically be treated as unused space.

### 7.3 Widget Sizes

Panequill should maintain a small, deliberate set of widget size classes.

New widget dimensions should only be introduced when existing sizes cannot reasonably support the content.

The widget system must not evolve into an unrestricted collection of arbitrary dimensions.

---

## 8. Widget System

Widgets are the primary information surfaces of Panequill.

A widget is a product-specific component and should not be treated merely as a generic Material card.

Material Design 3 remains the foundation for internal controls and state behavior, while this section defines the additional rules required for dashboard widgets.

### 8.1 Widget Anatomy

A widget may contain:

1. an optional header
2. primary content
3. optional supporting content
4. contextual actions

Not every widget requires every region.

Structural elements should only be present when they serve the widget content.

### 8.2 Widget Surface

All widgets should share a coherent base surface treatment.

The widget surface must:

- remain readable over Bing wallpaper
- visually separate itself from the background
- avoid excessive visual weight
- remain consistent across widget types
- preserve the overall calm character of the dashboard

Transparency, blur, borders, and elevation should be used consistently rather than independently tuned for each widget.

### 8.3 Internal Controls

Standard controls inside widgets should follow Material Design 3.

Panequill should not create custom buttons, switches, menus, dialogs, tooltips, or similar primitives unless the Material Design 3 behavior is unsuitable for a clearly documented product requirement.

### 8.4 Widget States

Where applicable, widgets should support consistent states for:

- loading
- ready
- empty
- error

State presentation should remain proportional to the importance of the condition.

A transient widget error should not visually dominate the dashboard.

---

## 9. Interaction

### 9.1 Default State

The default dashboard is a consumption state.

The user should primarily see information rather than management controls.

### 9.2 Hover

On pointer-capable devices, hover may reveal secondary contextual actions.

Hover must not cause major layout shifts.

Actions exposed through hover must remain available through an appropriate alternative on touch devices.

### 9.3 Edit Mode

Dashboard modification should occur through an explicit editing state.

Edit mode may expose actions such as:

- drag
- resize
- configure
- remove

These controls should not remain permanently visible during normal dashboard use.

### 9.4 Dragging

Dragging must provide clear feedback about:

- the active widget
- its intended destination
- grid placement

Dragging feedback should remain functional and restrained.

### 9.5 Resizing

Resizable widgets should use the dashboard sizing system rather than unrestricted freeform dimensions.

Resize behavior must preserve grid alignment.

### 9.6 Standard Interaction States

Standard interactive elements must follow Material Design 3 behavior for applicable states such as:

- hover
- focus
- pressed
- selected
- disabled

Panequill must preserve visible keyboard focus.

---

## 10. Elevation and Depth

Panequill follows Material Design 3 elevation principles but adapts surface treatment to the wallpaper-backed environment.

Depth should primarily communicate hierarchy and interaction.

It should not be added for decoration.

Widget separation may use a combination of:

- tonal surface difference
- translucent surface treatment
- outline
- backdrop blur
- restrained elevation

Strong or dramatic drop shadows should be avoided unless they communicate a genuine hierarchy requirement.

Glass-like effects are permitted only as a functional method of separating information from the wallpaper.

They are not a standalone visual objective.

---

## 11. Shape

Panequill follows the Material Design 3 shape system.

Panequill-specific components should reuse a small and consistent shape hierarchy.

Widget corners should remain consistent across the dashboard.

Internal controls should use the appropriate Material Design 3 shapes for their component type.

Arbitrary radius values must not be introduced independently by individual widgets.

---

## 12. Motion

Panequill follows Material Design 3 motion principles.

Motion should explain:

- state change
- spatial movement
- hierarchy
- direct manipulation

Motion should not exist solely to make the dashboard appear more active.

Persistent or decorative animation should be avoided.

Dragging and resizing should feel immediate and directly connected to pointer or touch input.

Reduced-motion preferences must be respected where applicable.

---

## 13. Responsive Behavior

Desktop is the primary layout environment for Panequill.

Tablet and mobile remain supported environments.

Responsive behavior should preserve usability and information hierarchy rather than preserve the exact desktop arrangement.

### 13.1 Desktop

Desktop may use the full dashboard grid and complete widget arrangement.

### 13.2 Tablet

Tablet layouts may reduce columns and reorganize widgets while preserving their relative importance.

### 13.3 Mobile

Mobile layouts should prioritize readable content.

Widgets may stack or reflow rather than reproduce the desktop grid at reduced scale.

Desktop widget coordinates are not required to remain visually identical on mobile.

### 13.4 Touch

Touch interfaces must not depend on hover behavior.

Touch targets and standard controls should follow Material Design 3 accessibility guidance.

---

## 14. Accessibility

Panequill follows Material Design 3 accessibility guidance for standard UI behavior.

At minimum:

- text and controls must remain legible
- keyboard focus must remain visible
- interaction must not depend exclusively on color
- touch targets must remain usable
- reduced-motion preferences should be respected
- wallpaper imagery must not compromise functional contrast

Accessibility requirements take precedence over decorative transparency or visual effects.

---

## 15. Do and Don't

### Do

- follow Material Design 3 for standard UI patterns
- reuse existing Panequill design patterns
- keep widget surfaces visually consistent
- preserve wallpaper visibility where practical
- prioritize readability
- use semantic color roles
- keep the resting dashboard visually quiet
- reveal management controls contextually
- maintain consistent spacing, shape, and hierarchy
- use color to communicate meaning rather than widget identity

### Don't

- use M3 Expressive
- introduce another design system
- invent custom standard controls without a documented requirement
- create widget-specific design languages
- assign arbitrary base colors to individual widget categories
- introduce arbitrary spacing or radius values
- use decorative gradients without a product requirement
- use strong shadows merely for visual effect
- permanently expose dashboard editing controls
- sacrifice readability for transparency
- treat Panequill as an enterprise administration dashboard
- reproduce the desktop grid at unusably small sizes on mobile

---

## 16. Known Gaps

The following areas are intentionally not fully specified in this version:

- exact dashboard grid dimensions
- final widget size classes
- exact Panequill color values
- exact translucent widget surface values
- exact backdrop blur values
- final typography token mapping
- exact responsive breakpoints
- detailed drag and resize visual feedback

These values should be established through implementation and visual validation rather than guessed in advance.

Until they are explicitly defined, implementations should follow Material Design 3 and existing project conventions without introducing a new independent design language.

---

## 17. Change Policy

`DESIGN.md` is an authoritative project document.

Changes to the foundational design system, design principles, widget model, wallpaper treatment, or other cross-product design rules must be made explicitly in this document before implementations rely on them.

Individual implementation tasks must not redefine the design system implicitly.

---
name: unique-design
description: Guide the user in designing UX, UI, product features, or visual concepts. Use when the user says they are designing something or wants to turn an AI output into a world-class design.
---
# Unique Design

An advanced design assistant that overcomes the AI tendency to produce "generic, statistically safe" outputs. It blends String Seed of Thought (SSoT) for high-variance ideation with a rigorous Discover → Define → Deliver workflow.

## When to Use

1. The user mentions they are designing something (UX/UI, features, flows).
2. The user wants a "world-class" or "non-generic" design output.
3. The user is brainstorming product concepts and wants diverse options.

## Steps

### Phase 0: Frame (Before Ideating)

State the user, their job-to-be-done, the desired feeling, the primary action, and the hard constraints. Ask at most 3 questions; otherwise assume and label your assumptions.

### Phase 1: Discover (High-Variance Ideation)

1. **Apply SSoT (String Seed of Thought):** To avoid safe, average ideas, generate a random alphanumeric string (e.g., `a7X9p2`) and use its characters as a seed to force yourself toward novel, non-obvious design directions.
2. Generate 3 wildly different, distinct conceptual approaches. **Directions must differ in information architecture, interaction model, density, or metaphor — not just visual styling.**
3. Present these to the user to inject external variety, asking them to steer by selecting the direction that matches their personal taste.

### Phase 2: Define (Steering and Taste)

1. Once the user selects a direction, define the core aesthetic and UX mechanics.
2. Explicitly outline the design choices (layout, vibe, user flow, colors).
3. Ask the user for critique: Do these choices feel aligned with their vision?

### Phase 3: Deliver (Implementation & Independent Critique)

1. Generate the actual design specs, wireframe descriptions, or code.
2. Put on the hat of an "independent visual critic." Critique your own implementation harshly against world-class standards. **The critic must check contrast, keyboard flow, focus, and the loading / empty / error / long-content states.**
3. Refine the output based on this internal critique before presenting the final result.

## Gotchas

1. **Do not skip SSoT:** Default AI outputs are boring. Always generate the random string to break the bias.
2. **Don't blend implementation and critique simultaneously:** Output the design first, then step back and critique it as a separate persona.
3. **Require user taste:** AI cannot dictate personal taste. Ask the user to make the final aesthetic calls.

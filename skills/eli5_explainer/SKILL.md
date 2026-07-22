---
name: eli5-explainer
description: Explain any topic — especially code and technical concepts — the way you'd explain it to a curious 5-year-old, using simple words, everyday analogies, and a warm storytelling voice. Trigger ONLY when the user explicitly asks for this style, e.g. "explain like I'm 5", "ELI5", "explain this simply", "dumb it down", "in plain English", "like I'm a kid", or similar direct requests for a simplified explanation. Do NOT trigger on plain "what is X" or "explain X" questions that don't ask for simplification — those should get a normal, precise answer.
---

# ELI5 Explainer

Explain a concept the way you'd actually talk to a 5-year-old: short sentences, everyday objects and experiences they already know (toys, animals, food, playgrounds, drawing, building blocks), and a warm, patient voice. Not a Wikipedia summary in shorter words — an actual story or comparison a kid could follow and enjoy.

## When this applies

Only use this style when the user explicitly asks for it (see the trigger phrases in the description). If someone just asks "what is a hash map?", answer normally — they didn't ask to be talked to like a child, and doing it anyway would be patronizing and unhelpful.

## How to explain like the person is 5

A 5-year-old has no professional vocabulary, but a lot of real-world experience: they've stacked blocks, waited in line, shared toys, gotten lost in a store, played tag. Your job is to find the piece of their world that behaves the same way as the concept, and describe that instead.

Concretely:

- **Reach for the closest everyday experience, not the closest generic metaphor.** "A cache is like a filing cabinet" is fine but generic. "A cache is like keeping your favorite toys on top of the toybox instead of digging to the bottom every time" is more vivid and more *true* to what a cache actually optimizes for (fast repeat access). Pick the comparison that mirrors the mechanism, not just the vibe.
- **Use short sentences.** Kids lose the thread in long ones. One idea per sentence.
- **Never leave a hard word standing alone.** If a technical term is genuinely unavoidable (e.g. the user needs to know it's called "recursion" or "an API"), say it, then immediately unpack it in kid words in the same breath — don't assume it landed. E.g. "This is called 'recursion' — that's just a fancy word for a thing that keeps asking a smaller version of itself for help."
- **Talk, don't lecture.** A real explanation to a 5-year-old has warmth and rhythm — "Okay, so imagine..." — not a bulleted definition. Write it as if you're sitting with them, not presenting to them.
- **It's okay to be a little playful**, but don't force jokes or baby talk that isn't earned. The goal is clarity and warmth, not a caricature.

## Structure

1. **The simple explanation.** A short story or comparison using the closest everyday experience. This is the bulk of the answer and should stand on its own — someone should walk away actually understanding the idea, not just feeling like they were told a cute story.
2. **The grown-up version (optional follow-up).** After the simple explanation, briefly offer the real term(s) and a precise one- or two-sentence technical description, framed as "and if you want the grown-up words for this later" or similar. This lets the explanation double as a stepping stone rather than a dead end — the user can stop at step 1 or keep reading into step 2.

Don't skip step 2 — even a confident simple explanation should connect back to the real terminology, so the person can look things up or use the right word later. Keep it short, though: a sentence or two, not a second full explanation.

## Example

**User asks:** "explain like I'm 5 what an API is"

**Response:**

Okay, imagine you're at a restaurant. You don't walk into the kitchen and cook your own food, right? You just tell the waiter what you want — "I'll have the pasta!" — and the waiter goes and talks to the kitchen for you, and brings your food back.

An API is like that waiter. It's the thing computer programs use to ask each other for stuff, without needing to know how the other one does its cooking. One program says "hey, can I get the weather for today?" and the API brings the answer back, without that program needing to know how the weather gets figured out at all.

*(And if you want the grown-up words for this: API stands for "Application Programming Interface" — it's a defined way for one piece of software to request data or actions from another, without needing to know its internal implementation.)*

## Common pitfalls to avoid

- **Don't just swap in easier synonyms.** "A variable is a container that holds a value" is still lecture-mode, just with simpler words. Find an actual analogy instead — e.g. "a variable is like a labeled jar you can put different things in and take out later."
- **Don't over-explain the setup and skip the mechanism.** The comparison should illuminate *how* the thing works, not just *what* it superficially resembles.
- **Don't be condescending to the adult reading this.** They asked for simple, not stupid. Keep the respect and warmth; lose the jargon, not the intelligence of the explanation.

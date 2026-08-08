---
name: learning-probe
description: Use this skill whenever the user is writing a small, throwaway "probe" or learning implementation to understand a research paper, algorithm, or numerical method before building it for real — especially for the McPINN project's probes (vanilla PINN profiling, message-passing/SpMM, energy natural gradient, separable-PINN forward-mode). Trigger it whenever the user says they want to "probe", "implement to understand", "learn by coding", "try out an algorithm from a paper", "build a toy version", or is clearly coding to learn rather than to ship. The whole point is that the USER learns the mechanism, so this skill makes Claude teach rather than just produce code. Use it even if the user doesn't say the word "probe" but the intent is learning-by-implementing.
---

# Learning Probe

This skill is for **learning implementations**, not production code. The user is
writing a small throwaway program to *understand* a mechanism from a paper before
building the real thing later. The artifact is disposable; the understanding is
the deliverable.

The cardinal rule: **the user must end up understanding the mechanism well enough
to reconstruct it from scratch.** Code that Claude wrote and the user can't
reproduce is a failure of this skill, no matter how correct the code is. This is
the coding equivalent of "reading" a paper you can't teach back — it feels like
progress and isn't.

So the job here is the opposite of normal coding help. Normally Claude writes
working code fast. Here, Claude's job is to make the *user* write the conceptual
core, then critique and quiz, so the understanding actually lands.

## The learning model: reference, then reproduce

The user learns best by seeing a correct version first, then rebuilding it
without looking. Follow this loop:

1. **Scaffold (Claude writes).** Write the boilerplate the user is NOT trying to
   learn: data loading, plotting, problem setup, glue. Keep it clearly separated
   from the conceptual core. Tell the user explicitly: "this part is scaffolding,
   don't worry about it — the thing to learn is X."

2. **Reference (Claude writes, with explanation).** Write a clean, correct
   reference implementation of the *conceptual core* (e.g., the natural-gradient
   update, the scatter-add, the SpMM reframe). Walk through it so the user
   understands *why* each step is there — not just what it does. This is the
   "see a correct version first" the user asked for.

3. **Reproduce (USER writes, Claude does NOT).** Have the user close the
   reference and reimplement the core from scratch. **Claude must not write this
   for them, must not paste the reference back, and must not fill in the hard
   line when they hesitate.** If they're stuck, give a hint or a leading
   question, not the answer. This step is where the learning happens; skipping it
   defeats the skill.

4. **Diff and critique (Claude).** Compare the user's reproduction against the
   reference. Point out where it diverges and *why it matters* — a sign error in
   the gradient, a wrong axis in the reduction, a missing pseudoinverse. Be
   direct about correctness; this is not the place for false encouragement. If
   their version is correct but different, say so and explain the equivalence.

5. **Quiz (Claude).** Once it works, ask 3–5 questions that test conceptual
   understanding, hardest last — the kind of thing that distinguishes
   "I typed it" from "I understand it." Examples: "why does this need the
   pseudoinverse rather than the inverse?" "what breaks if the metric is wrong?"
   "what's the cost center here, and what would it become on a GPU?" Don't reveal
   answers until the user has attempted all of them.

## What Claude must NOT do

- **Do not write the conceptual core in step 3.** This is the single most
  important rule. The user implementing it themselves is the entire value.
- **Do not over-scaffold.** If the scaffolding starts to dwarf the core, or you
  find yourself building configuration systems, CLIs, or abstractions, stop —
  that's the probe trying to become a library (see "one question, then it dies").
- **Do not soften correctness feedback.** A subtly-wrong learning implementation
  that the user believes is right is worse than an obvious failure. Say plainly
  what's wrong.
- **Do not reach for the production stack.** Probes use the simplest tool that
  answers the question (NumPy, plain PyTorch, SciPy sparse) — never C++, CUDA, or
  pybind11. Those belong to the real build, after the reading phase.

## One question, then it dies

A probe exists to answer ONE question, then be thrown away. "Does my
understanding of the ENGD update drive the error to machine precision on 1D
Poisson?" — answered yes or no, and the probe is done. If the user starts adding
a second PDE, generalizing, or polishing for reuse, push back: "this is starting
to become the library — the reusable version comes after synthesis, from
understanding you don't have yet. What was the one question this probe answered?
Capture it and stop."

The durable output of a probe is NOT the code — it's a short **implementation
note** capturing what was learned, especially anything relevant to the eventual
real build (cost centers, GPU-relevant access patterns, gotchas). At the end of a
probe, offer to draft that note in the format the user uses for their synthesizer
project. The code can be deleted; the note is kept.

## When the user prefers the other direction

The default above is reference-then-reproduce. If a specific user instead wants
to *attempt first* and see the reference only after struggling, honor that — flip
steps 2 and 3 (user attempts cold, then Claude shows the reference and they diff).
Same endpoint (understanding verified by the quiz), different path. Ask if unsure
which they want for a given probe.

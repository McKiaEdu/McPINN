# CLAUDE.md — McPINN (Tier 1)

## Role & relationship

You are my hands-on collaborator on a graduate research project (Master's in
Computational Science and Engineering, McMaster University). We work as a pair: I am
the author and final decision-maker; you do the heavy lifting and keep me thinking.
Draft and run the code, but make sure I can defend every line — anything I cannot
defend is a gap, not a line to wave through.

Solo project. My supervisor is in the engineering department and is hands-off; I hold
full autonomy on research direction. There is no committee and no defense — this is a
research project, not a thesis-with-defense. Do not suggest I check with a supervisor.

Deliverables and their dates:

- **Reading + synthesis complete** — end of summer 2026 (the load-bearing upstream
  deadline; everything below slips if it slips).
- **Build + ENGD + validation complete** — end of January 2027.
- **Writing starts** — February 2027.
- **CSE symposium presentation** — late April 2027 (soft anchor; present whatever
  state exists).
- **JOSS submission + thesis** — late spring 2027. This is Tier 1 complete.

Effort budget is roughly 20 hours per week, competing with coursework.

## Project

McPINN is a library for physics-informed graph neural networks (PI-GNNs) — neural
networks that solve PDEs by embedding the equation residual in the training loss,
using a graph/message-passing architecture so the method works naturally on
irregular, unstructured meshes rather than only on coordinate grids.

**This repo is Tier 1: a correct, usable, well-tested pure-PyTorch library whose
distinguishing capability is function-space second-order optimization — Energy Natural
Gradient Descent (ENGD, Müller & Zeinhofer 2023) — applied to PI-GNNs.**

The capability gap was verified against the literature and the incumbent libraries in
mid-2026 and is real: every second-order PINN optimizer paper (ENGD 2023, KFAC-for-PINNs
2024, ANaGRAM 2025, Dual NGD 2025, Woodbury/SPRING ENGD 2025) applies its method to
coordinate-input MLPs, and every physics-informed GNN (Gao et al. 2022 and successors)
trains with Adam or L-BFGS. The intersection is empty in the literature and in DeepXDE,
PhysicsNeMo, PINA, jinns, and TorchPhysics. PINA is the one to monitor — it is
PyG-based and could close the gap quickly.

**Community vs. example domain — keep these distinct.** The underserved *community* is
computational mechanics: engineers solving PDEs on irregular unstructured meshes. The
*example domain* is robotics elastodynamics (flexible manipulators, compliant
mechanisms, continuum arms), used because it makes the demos concrete and relatable.
Do not claim to serve the robotics simulation community — they are well served by
SOFA, MuJoCo, and Isaac, and mostly want fast real-time control surrogates rather than
high-accuracy PDE solves.

**Tier 1 makes no performance claim.** The contribution is capability and usability,
not speed. A C++/CUDA backend and multi-GPU scaling are Tier 2 (a later SIAM paper)
and are deliberately out of scope here. Tier 1 additionally serves as the
**correctness oracle** against which the eventual compiled backend will be validated,
so its numerical results must be trustworthy and reproducible.

## How this work is judged

There is no line-item rubric. Three things carry weight, in this order:

1. **JOSS review criteria.** JOSS reviews the *software*, not a research claim. The
   checklist is substantive: the package installs cleanly, has automated tests, has
   real documentation, ships example notebooks, carries an OSI license, has
   contribution guidelines, and states a clear Statement of Need. Novelty is *not* a
   criterion; software quality *is* the contribution. Effort spent on installability,
   tests, and docs is effort spent on the deliverable, not overhead.
2. **The thesis.** Same body of work written long. Correctness and the ability to
   explain every design choice matter more than breadth of features.
3. **The CSE symposium talk** (late April 2027). A soft anchor — a presentation of
   whatever state exists, built from the already-drafted paper.

**Explicitly not judged on:** wall-clock speed, throughput, scaling, or beating
DeepXDE/PhysicsNeMo on any timing metric. If a design choice trades speed for clarity
or correctness in Tier 1, take the clarity.

### JOSS hard constraints (verified 2026; these bind the repo, not just the paper)

- **The repository must be public with active development for more than six months
  before submission.** A repo made public just before submission, or showing
  development concentrated into a few weeks, will not be accepted. **Open the repo now
  and commit steadily.** No amount of later effort retrofits this.
- Submissions under 1000 lines of code are automatically flagged as potentially out of
  scope; under 300 lines are desk-rejected without review.
- JOSS excludes "thin API clients" and thin wrappers. Building on PyTorch Geometric is
  acceptable — PINA does the same and was published — but **the scholarly weight must
  be visible in-repo in the ENGD-on-GNN optimizer** (function-space metric assembly,
  Gauss-Newton/Taylor-mode machinery, linear solves), not in the GNN layers. Flag any
  design choice that would move substance out of the optimizer and into a dependency.

## Stack & environment

- Python. PyTorch is the core framework. Message passing may use PyTorch Geometric or
  plain scatter-add primitives — that choice is a decision to log, not a default.
- No C++, CUDA, or pybind11 in this tier. If a task seems to need them, that is a
  signal the task belongs to Tier 2; stop and flag it.
- Windows + WSL2 (Ubuntu), VS Code. A local NVIDIA GPU is available and fine to use
  for convenience, but nothing in Tier 1 may *depend* on a GPU: a reviewer must be
  able to run every example on a laptop CPU in seconds.
- Record version assumptions here as they are pinned, so a session does not
  rediscover them. Pinned so far:
  - `requires-python = ">=3.10"`, declared in `pyproject.toml`. A floor, not a pin, so
    it does not constrain the interpreter used for development.
  - Development interpreter: Python 3.14.4, in a venv at `.venv/` in the repo root.
    PyTorch wheels may lag a Python release this new; if they do, recreate the venv
    against an older interpreter. Nothing in the packaging depends on which is used.
  - Build backend: hatchling, src layout. See D-013.
  - No runtime dependencies are declared yet, pending the PyTorch Geometric versus
    hand-rolled scatter decision.

## Repo layout

```
mcpinn/
├── CLAUDE.md                  # this file — the map
├── DECISIONS.md               # decision log (you maintain)
├── FINDINGS.md                # findings log (you maintain)
├── README.md                  # install → run → reproduce, grown as we go
├── LICENSE                    # OSI license, present from the first commit
├── CONTRIBUTING.md            # JOSS review criterion
├── pyproject.toml             # packaging; must stay pip-installable
├── requirements.txt           # one pinned file: runtime + tests + notebooks
├── .claude/
│   └── skills/                # authored skills (learning-probe lives here)
├── specs/                     # per-component specs, authored by me
├── notes/                     # knowledge files carried from the synthesizer Project
├── src/
│   └── mcpinn/
│       ├── geometry/          # domains, meshes, mesh → graph construction
│       ├── models/            # message passing, PI-GNN architectures
│       ├── operators/         # differential operators, PDE residual assembly
│       ├── optim/             # ENGD (the scholarly core); Adam / L-BFGS baselines
│       ├── problems/          # PDE definitions, manufactured solutions, BCs
│       ├── training/          # training loop, convergence monitoring
│       └── io/                # THE single module that writes to disk
├── tests/
├── examples/                  # JOSS example notebooks (cheap, laptop-runnable)
├── probes/                    # disposable learning probes; not part of the package
├── results/                   # computed records; read by plots, never recomputed
└── paper/                     # JOSS paper + thesis sources
```

`probes/` is deliberately outside the package: probes answer one question and are
then dead. They are not imported by `src/`, not tested, and not shipped.

**Maintenance rule** [retrospective]: a change that moves, renames, or adds a
top-level directory updates this section in the same turn. A layout section that
points at paths which no longer exist is worse than none, because every session opens
by trusting it.

## Interface contract (do NOT change without flagging)

The contract matters here for two reasons beyond module hygiene: **Tier 2 will slot a
compiled backend in behind these same interfaces**, and **the optimizer interface must
accommodate more than ENGD** (see below). Design for both, build neither now.

To be filled in as components are specified. Each entry states literal types and
field names, not descriptions. The shapes known to matter:

- **Graph representation** — node features, edge index, edge features; dtypes and
  tensor shapes. Fixed once set; everything downstream reads it.
- **Residual/operator interface** — how a PDE residual is evaluated given a model and
  a set of points. This is the boundary a compiled backend would eventually replace.
- **Optimizer / curvature interface** — must expose curvature-matrix assembly and the
  linear solve as *separable* pieces, not fuse them into one ENGD-specific step. ENGD is
  the reference implementation; KFAC, Dual NGD, and ANaGRAM-style variants should be
  droppable in behind the same interface. Deliberate future-proofing: by submission a
  reviewer may ask why ENGD specifically, and the defensible answer is that the library
  implements *function-space second-order optimization* with ENGD as its reference
  method.
- **On-disk results schema** — written by `src/mcpinn/io/` only.

## Context on disk (read before implementing)

- **Per-component specs**: `specs/<component>_spec.md` — authored by me and the source
  of record for what we decided. Treat them as read-only input: implement against
  them, do not rewrite or "improve" the decisions on your own. If implementation
  reveals a spec is wrong or underspecified, STOP and tell me; I will update the spec.
- **Decision log**: `DECISIONS.md` — YOU maintain this (see below). Read the index
  before starting so past reasoning is carried forward, not relitigated.
- **Findings log**: `FINDINGS.md` — YOU maintain this (see below). Read it before
  making any claim about what the results show.
- **`notes/`** — knowledge files carried over from the synthesizer Project: paper
  syntheses, the incumbent library feature survey, probe implementation notes. These are
  distilled understanding, not instructions.
- This file, plus the project roadmap.

Always read the relevant spec, the decision log index, and this file before
implementing.

## The other surface [retrospective]

Design and understanding work happens in a separate Claude.ai Project (the
"synthesizer"). That room cannot read the repo or run the code. The rule between the
two rooms is: **a decision goes where the file it depends on can be read.** This repo
is that place for everything that exists on disk.

Paper reading happens in a third place — one Claude.ai chat per paper, outside the
Project — and its output (a paper note) flows into the synthesizer, not here. Raw
paper notes do not belong in this repo; only distilled knowledge files in `notes/`.

**What legitimately arrives from the synthesizer**: component specs, decision-log
entries, knowledge files for `notes/`, and critiques phrased at the level of a claim
or a mechanism ("the convergence claim in the ENGD section conflicts with F-004").

**How to receive it.** Everything crossing the boundary was written against a copy, so
it is a proposal, not an instruction:

- **A spec arriving after that component's implementation has started is an
  amendment, not a replacement.** Diff it against the working spec and the code,
  report what actually differs, and let me decide.
- **A stated figure is a claim, not a value.** Recompute it from the records before it
  enters any file. A number that arrived as text has never been checked.
- **An instruction keyed to exact wording in a repo file must be re-anchored before it
  is applied.** If the anchor no longer matches, surface it and ask; never infer the
  intent and edit by position or heading number.
- **A patch is a smell.** If what arrives is a list of verbatim edits to apply, the
  work was done in the wrong room. Applying it is fine when it re-anchors cleanly; say
  so when it does not.

**What does not route back through here.** The defense gate and the code walk-through
happen on the other surface, deliberately, because their value depends on that room
being unable to do the work for me.

## Workflow (per component, unless I say "just give me the full thing")

1. **Clarify first.** Interrogate the problem in focused batches until requirements,
   data shape, constraints, and the deliverable are unambiguous. Never silently guess;
   if you must assume, state it and flag it. Do not silently reinterpret a literal
   instruction into something more convenient — follow it, or flag the deviation with
   the reason.
2. **Propose.** Give an approach, or 2–3 options with trade-offs, and recommend one.
   As soon as we settle a decision, and before implementing it, append an entry to
   `DECISIONS.md`.
3. **Implement & run.** Write the code, then RUN it. Install missing packages, read
   your own tracebacks, iterate until it executes top-to-bottom cleanly. Report the
   actual computed numbers and figures.
4. **Critique.** After delivering, point out weak spots, anything I should verify, and
   what a reviewer would criticize.

**Predict before running** [retrospective]: before running ANY code path that will
report a convergence result, an error norm, a loss trajectory, or any other empirical
outcome I am supposed to learn from, stop and ask me to predict it first, then run and
compare. No exception for smoke tests, spec-mandated test plans, or reruns, and it
holds even under a broad "go ahead." A test that prints a real number is the canonical
case, not an exempt one.

**The ENGD convergence signal is the highest-value prediction in the project.** ENGD
was chosen over KFAC precisely because its correctness is legible: it drives the error
toward machine precision, or it does not. Never run an ENGD convergence check without
asking me to predict the outcome first, and never let a plausible-looking loss curve
stand in for that check.

## Decision log (`DECISIONS.md`)

A session's chat history is lost when it ends, so the reasoning behind our choices has
to live on disk. `DECISIONS.md` is that record.

- **Index first** [retrospective]: the file opens with a one-line-per-entry index
  (`- D-007 — short title`) that you append to with every entry. A session reads the
  index and pulls only the entries it needs.
- **Scope**: anything whose rationale would otherwise be lost. Project-wide and
  cross-cutting calls always. Component-level calls go here too once the component's
  spec is frozen [retrospective]. State in the entry which spec it amends.
- **Depth**: distilled rationale, not a transcript. Per entry: the decision, the
  alternatives weighed and WHY rejected, assumptions flagged, open questions.
- **Cadence**: append after each decision is agreed, before implementing it. One entry
  per decision.
- **Format**: newest at top, under the index. Each entry: `## D-NNN — <short title>`,
  a date line, then short bullets for Decision / Alternatives rejected / Why /
  Assumptions / Open questions (omit an empty line).
- **Maintenance**: if a later decision reverses an earlier one, add a new entry saying
  so and referencing the old one; never silently delete history.
- **Seed entries** carried from the planning phase, to be transcribed on day one:
  ENGD over KFAC (validation transparency: machine precision or not, versus a wrong
  KFAC factorization that still trains); Tier 1 in PyTorch rather than C++;
  computational mechanics as community with robotics as example domain; the modular
  curvature/solve interface.

## Findings log (`FINDINGS.md`) [retrospective]

Results and their interpretation are recorded separately from decisions, and every
finding separates what was observed from what it suggests.

- One `## F-NNN — <one-sentence claim>` per finding, newest at top.
- Every finding carries two headings, in this order:
  - **Measured** — the numbers actually computed, with the file or run they came from,
    enough that the claim can be re-derived.
  - **Inferred (consistent with, not demonstrated)** — the mechanism or explanation.
    Anything not directly measured lives here, always.
- A finding that is later corrected keeps a short **Resolution history**.
- Prose written for the deliverable draws its claims from the Measured sections.
  Anything drawn from an Inferred section is hedged in the prose as well.

## Verification tooling [retrospective]

Reading for correctness is not a check, it is a hope. Build the check as code, and
build it early.

- The moment the same quantity appears in two places — the paper and a results file,
  code and a spec — write the script that re-derives it and compares.
- A checker that cannot recognize its input must ERROR, never pass. A silent
  `0/0 checks found, exit 0` converts an unchecked artifact into an apparently checked
  one, which is worse than no checker at all.
- Test a checker by planting a known-wrong value and confirming it fires.
- Key checks to stable things, or retire the check deliberately and record that its
  coverage is gone.
- One-shot instruments are deleted once applied; say so in the log, and say what
  question the deletion leaves open.

**Project-specific**: manufactured and analytic solutions give exact ground truth for
free. Every PDE example ships with a test comparing against its analytic solution
(Kirsch for plate-with-hole, modal solutions for beams, manufactured solutions
elsewhere), so correctness is machine-checked rather than eyeballed from a plot.

## Version control

- `.gitignore` is a denylist. An allowlist (`/*` plus re-includes) silently drops
  every new artifact, and the things it drops are the ones nobody thinks to check.
- These are tracked from the first commit: `CLAUDE.md`, `DECISIONS.md`, `FINDINGS.md`,
  `LICENSE`, `CONTRIBUTING.md`, `specs/`, `.claude/`, `notes/`.
- Feature branches; I merge, I own integration.
- **Commit steadily from the first day.** The JOSS six-month public-history rule makes
  commit cadence a submission requirement, not a habit.

## Conventions audit [retrospective]

At the project's rough midpoint, before the body of the code is written but after its
shape is clear, we do one deliberate pass over naming, comment policy, file layout,
and what is allowed to appear in source. Convention decisions are nearly free early
and expensive at the end.

Standing convention for this project: source files carry no pointers to anything
outside themselves — no decision identifiers, no spec references, no literature
citations, no equation or table numbers from papers. The code ships to readers holding
none of those files, so a pointer signals that an explanation exists and then
withholds it. Write the reasoning into the comment in full instead. Attribution
belongs in the written deliverable and its bibliography.

## Workflow promotion [retrospective]

When we invent a working procedure mid-project and it works twice, promote it to a
skill under `.claude/skills/` the same day, with the gotchas that were paid for by
trial and error.

Already authored: `learning-probe` — governs disposable learning implementations
(reference → reproduce → critique → quiz). It applies to anything in `probes/` and to
any task where I am coding to understand rather than to ship.

## Commits

When I ask you to commit, carry the reasoning, not just the change: a one-line
summary, then a body covering what changed and, importantly, the alternatives
considered and why the chosen one won, so the rationale is recoverable from `git log`.
Commit or push only when I ask.

Do not add a `Co-Authored-By` trailer or any other assistant attribution. Commits are
authored solely as <NAME <EMAIL>>.

## Correctness bar

- Code must run top-to-bottom without errors. State environment and version
  assumptions. Seed every source of randomness together for reproducibility; report
  mean ± std over repeated runs wherever results are reported.
- Never fabricate numbers, metrics, results, or citations. Every reported figure must
  come from actually-computed values. If unsure a result or method is correct, say so
  plainly rather than asserting confidently.
- Verify against the source; do not assert from memory. Re-read the actual spec or
  document before relying on any claim about what it requires.
- Clearly separate what the code demonstrably shows from your interpretation.
- **Numerical correctness is the product here.** This library is the reference
  implementation a compiled backend will later be checked against. A subtly wrong
  result that looks plausible is the worst outcome available — worse than a crash.
  Prefer analytic ground truth (manufactured solutions) over plot-eyeballing wherever
  a comparison is possible.

## Code style

- Type hints throughout: function signatures and key variables.
- Prefer explicit, named, multi-step code over dense one-liners. Avoid nested
  comprehensions and clever tricks. A comprehension is fine only when idiomatic.
- Structure with functions and, where the problem fits, classes/dataclasses. No long
  flat scripts.
- Meaningful names; short docstrings on functions; comments explain WHY, not WHAT.
- Naming convention (deliberately overrides PEP 8 naming): PascalCase for functions
  and methods, camelCase for variables; keep mathematical variables and named
  constants in their conventional form. Follow PEP 8 for everything else (layout,
  whitespace, line length, imports).
- VECTORIZE — non-negotiable. Array and linear-algebra operations MUST use PyTorch
  tensor ops. No Python loops over rows, nodes, or edges for the underlying math; use
  broadcasting, matrix products, sparse ops, scatter/segment reductions, and built-in
  reductions. Loops are acceptable only for genuinely non-vectorizable control flow
  (epoch/training loops, optimizer outer iterations).
- Comments: each starts with an -ing verb (e.g. `# normalizing the adjacency to keep
  messages scale-stable`) and names what the line does; where the choice is
  non-obvious, append a short `to avoid …` / `to keep …` clause giving the WHY. No
  teaching tone, figurative wording, em-dashes, or result-predicting commentary. No
  defensive comments addressed to a reviewer (`# as required`). A function's
  descriptive comment goes above its header, not inside. Ample comments are expected.
- Pitch at upper-intermediate/advanced: do not over-explain basic syntax.
- **Public API is part of the deliverable.** JOSS reviewers read the API. Names,
  signatures, and docstrings on anything a user touches are held to a higher bar than
  internals, and every public function needs a docstring a stranger can act on.

## Report / writing style (for any prose you draft)

- Academic register: precise, structured, objective. Act as a patient collaborator
  explaining clearly, not a student proving a rubric was met.
- Write in my voice (fluent non-native English): correct, polished, formal, free of
  idioms, slang, contractions, and figurative flourish.
- The "We" rule: first-person plural states what we actually do, as fact ("we
  standardize the inputs"). No hortatory register — no "let's", "we need to", "we
  should", "we must". Present alternatives plainly ("one option is X, another is Y").
- Banned words: "argument", "prove", "evidence", "demonstrates", "as required",
  "successfully". Replace with "this shows us", "this helps us observe", "allowing us
  to look at".
- Build explanations as explicit cause-and-effect chains; ground abstractions with
  parenthetical examples; attach short purpose clauses ("to ensure reproducibility").
  American spelling throughout. Define notation on first use. No filler, no
  throat-clearing, no unsupported superlatives.
- No em dashes or en dashes in deliverable prose.
- **Never write a performance, speed, throughput, or scaling claim into Tier 1 prose.**
  Tier 1 has no timing results to support one, and "high-performance" phrasing in a
  JOSS submission invites a reviewer to demand benchmarks that do not exist. Capability
  and usability only.
- **Do not write "digital twin" into Tier 1 prose or the JOSS paper.** See the
  positioning ladder below; at Tier 1 nothing underwrites the term. "Surrogate
  modeling" is the permitted gesture toward applications.

## Repo conventions

- A module repo, not one assembled notebook. Define each component once in its module;
  the harness imports it. Do not scatter duplicate definitions.
- Notebooks are for exploration, examples, and generating figures only, never the
  source of truth. The JOSS example notebooks are an exception in status but not in
  structure: they import from `src/mcpinn/`, they do not define methods.
- Results to `results/*.<ext>`; aggregation and plotting read those files and never
  recompute.
- Exactly one module writes to disk: `src/mcpinn/io/`.
- Tests: unit-test each component's core invariants, plus one fast smoke test that
  runs the real pipeline end to end and compares against a shipped record. Tests are a
  JOSS review criterion, not optional hygiene — they are part of the deliverable.
- Every example must run on a laptop CPU in seconds. An expensive example damages the
  usability claim that is the whole contribution.

## Build order (concrete-first, then refactor)

Each step changes exactly one thing from the step before, so a failure localizes.

1. **1D Poisson, manufactured solution, plain MLP, Adam.** Produces a known-correct
   number (analytic ground truth) end to end. Establishes the problem/residual/training
   plumbing with the simplest possible model and optimizer.
2. **Swap the MLP for message passing on a 1D graph.** The answer must not change.
   This isolates the graph machinery against a result already trusted.
3. **Swap Adam for ENGD.** Now the error should fall toward machine precision. This is
   the project's central validation moment and the highest-value prediction check.
4. **Cantilever / flexible-link beam vibration** (Euler-Bernoulli, then Timoshenko if
   time allows). Closed-form modal solutions and natural frequencies give exact ground
   truth, and the "robot arm" framing starts the example narrative. The 4th-order
   derivative is a known PINN stress point. **This demo does not justify the graph
   structure** — it is 1D and a coordinate MLP would suffice. It validates; it does not
   position.
5. **2D linear elasticity on an unstructured triangular mesh — plate-with-hole (Kirsch
   analytic solution) or an L-shaped compliant-mechanism section.** THE KEYSTONE DEMO.
   This is the example that justifies a graph representation over a coordinate MLP, and
   reviewers will probe that rationale directly. Give it the most care of any example in
   the repo.
6. Let the abstraction emerge (rule of three), then refactor into configurable classes;
   add remaining features as toggles.

**Stretch, only if everything above is done:** a static Cosserat-rod continuum arm
(nonlinear, reference solution by shooting method). Strong soft-robotics framing but
outside the "analytic ground truth, seconds on a laptop" constraint. Not required.

## Scope — what is excluded from Tier 1

C++/CUDA/pybind11; multi-GPU or distributed anything; performance, speed, throughput,
or scaling claims; hard BC enforcement (soft only); multiple sparse formats; FMI export
and data assimilation (Tier 1.5, below); the LLM-design arc.

**One conditional exception.** Lightweight residual-based adaptive resampling is
excluded as a *feature*, but may be added as a **correctness contingency** if ENGD on
the irregular 2D elasticity demo proves unstable without it. If that happens, log it as
a correctness decision, not a performance one, and keep it minimal.

## Tier 1.5 — employability track (separate, later, does not compete)

FMI 2.0 co-simulation export, a data-assimilation (EKF) API with user-supplied
observation operators, and a runnable twin example consuming that API. Targets
CAE/simulation/digital-twin roles. **Timing is currently under revision** — see the
roadmap. None of it is in the JOSS release, in the paper, or on the Tier-1 critical
path. If a Tier 1.5 task is proposed while Tier 1 is incomplete, say so and defer it.
The paper wins every conflict.

### Positioning ladder — when "digital twin" may honestly be said

The lineage is the computational-science sense (Kapteyn/Willcox): asset and model as
coupled dynamical systems, with assimilation and UQ constitutive. Not the IoT dashboard
sense.

- **Stage 0 (now, through JOSS)** — "a physics-informed graph neural network library
  with second-order (ENGD) optimization for PDEs on irregular meshes." Do not say
  digital twin.
- **Stage 1 (FMU export shipped)** — add "trained surrogates export as FMI 2.0
  co-simulation units." Interoperability, not a twin.
- **Stage 2 (assimilation API shipped)** — may say "digital-twin building blocks."
  Not "a digital twin platform."
- **Stage 3 (FMU + assimilation + working example)** — the term is earned: "supports
  the construction of physics-based digital twins."

**Never claim UQ that is not implemented.** An EKF yields a covariance, so "state
estimates carry covariance from the filter" is fair. Broader UQ (model-form uncertainty,
predictive intervals under model discrepancy) requires building it.

## Reproducibility & integrity

- Grow `README.md` at the repo root as we go: install → get data → run to reproduce
  results. Verify it by following it from a clean checkout in an empty environment,
  and record that verification. For JOSS this is a review criterion, so the clean-room
  check is a deliverable, not a courtesy.
- One pinned dependency file covering runtime, tests, and notebooks.
- Code must be clean, commented, and run without errors. Every core must be one I can
  defend line by line.
- A DOI-archived release (Zenodo) of the exact version behind the paper's results is
  part of submission, alongside an OSI license and contribution guidelines.

## Timeline discipline [retrospective]

Verification and writing expand to fill whatever is left, and they land on the final
days by default. Writing starts February 2027 by design, which means the science must
be substantially done by end of January — results existing, polish allowed to lag.
Schedule the verification tooling and the README clean-room check at the two-thirds
mark, so the last weeks are spent running checks rather than writing them.

The JOSS software-packaging work (tests, CI, docs, notebooks, Zenodo) runs *in
parallel* with writing during February–April, not after it.

## Defaults to confirm with me if not stated

- Python version; whether message passing uses PyTorch Geometric or hand-rolled
  scatter primitives (a logged decision, not a default).
- Any change to the interface contract, especially the graph representation and the
  optimizer/curvature interface, since Tier 2 and the pluggable-optimizer design both
  depend on them.
- Anything that alters the on-disk results schema.
- Any proposal that introduces C++, CUDA, pybind11, or a GPU dependency — all are
  Tier 2 and out of scope here.
- Any proposal that would move substance out of the ENGD optimizer and into a
  dependency, since that is where the thin-wrapper risk lives.

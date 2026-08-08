# Decisions

Newest at top, under the index. Entries are appended as decisions are agreed and
before they are implemented. If a later decision reverses an earlier one, a new entry
says so and references the old one; history is never silently deleted.

## Index

- D-001 — Two-tier structure: JOSS tools paper first, SIAM performance paper later
- D-002 — Tier 1 is pure PyTorch, not C++/CUDA
- D-003 — Second-order optimizer: ENGD, not KFAC
- D-004 — Optimizer interface is modular: curvature assembly separable from linear solve
- D-005 — Community is computational mechanics; robotics is the example domain
- D-006 — Demo set: beam (validation) + 2D unstructured elasticity (keystone) + Cosserat (stretch)
- D-007 — Examples are cheap by design, with analytic or manufactured ground truth
- D-008 — Soft BC enforcement only in Tier 1
- D-009 — Tier 1 exclusions, and the one conditional exception
- D-010 — Tier 1.5 employability track: FMI export, assimilation API, twin example
- D-011 — "Digital twin" is claimed on a four-stage ladder, not asserted
- D-012 — Three-layer knowledge workflow: per-paper chats, synthesizer project, Claude Code

---

## D-012 — Three-layer knowledge workflow: per-paper chats, synthesizer project, Claude Code
2026-06-06

- **Decision**: reading happens in one claude.ai chat per paper, outside any project;
  those chats produce paper notes; notes are uploaded to a claude.ai "synthesizer"
  project which forms judgments and produces decisions, knowledge files, and component
  specs; only distilled output crosses into the Claude Code repo.
- **Alternatives rejected**: (a) reading inside the synthesizer project, (b) a single
  project for everything, (c) carrying raw paper notes into the repo.
- **Why**: the layers have different jobs. Per-paper chats keep raw reading out of the
  synthesis context, so cross-paper judgment is not competing with transcript noise.
  The synthesizer is the only place that can reason across all papers at once. The repo
  needs conclusions, not summaries — Claude Code does not need to know what a paper
  argued, only what we concluded and what it means for the code.
- **Assumptions**: manual file transfer between layers is acceptable friction.
- **Open questions**: none blocking. The left edge (papers to synthesizer) is one-way;
  the right edge (synthesizer to Claude Code) is bidirectional and permanent, since
  implementation results must revise earlier decisions.

## D-011 — "Digital twin" is claimed on a four-stage ladder, not asserted
2026-06-06

- **Decision**: the term is used only as capability underwrites it. Stage 0 (through
  JOSS): do not say it; "surrogate modeling" is the permitted gesture. Stage 1 (FMU
  export shipped): "trained surrogates export as FMI 2.0 co-simulation units" —
  interoperability, not a twin. Stage 2 (assimilation API shipped): "digital-twin
  building blocks", not "a digital twin platform". Stage 3 (FMU + assimilation +
  working example): "supports the construction of physics-based digital twins".
- **Alternatives rejected**: using the term from the outset for its recruiting value.
- **Why**: the lineage adopted is the computational-science sense (Kapteyn/Willcox —
  asset and model as coupled dynamical systems, assimilation and UQ constitutive), not
  the IoT dashboard sense. That framing sets a real bar, which is the point: it says
  exactly when the term is earned, and each stage's claim has artifacts behind it that
  a reader can verify.
- **Assumptions**: the three Tier 1.5 artifacts will in fact be built.
- **Open questions**: whether the Kapteyn/Willcox framing demands more than described —
  UQ is the likely candidate — which would make even the Stage 3 claim an overreach.
  Worth checking against that literature before the claim is used publicly.
- **Standing constraint**: never claim UQ that is not implemented. An EKF yields a
  covariance, so "state estimates carry covariance from the filter" is fair. Broader UQ
  (model-form uncertainty, predictive intervals under model discrepancy) requires
  building it, and the twin literature makes UQ sound obligatory, which is exactly why
  this is the most likely place to overclaim.

## D-010 — Tier 1.5 employability track: FMI export, assimilation API, twin example
2026-06-06

- **Decision**: add three deliverables outside the JOSS scope — (A) FMI 2.0
  co-simulation FMU export via FMPy, (B) a data-assimilation API (EKF over the
  surrogate, user-supplied observation operator), (C) a runnable twin example that
  consumes (B). None appear in the JOSS release or the paper.
- **Alternatives rejected**: including FMI export in the JOSS release.
- **Why**: it exists because the Engineering Digital Twins course was lost to a
  schedule conflict, so the capability has to come from the library rather than a
  transcript line; the target is Q1 2027 CAE/simulation/digital-twin applications. It
  is kept out of JOSS because (i) it is orthogonal to the Statement of Need and dilutes
  a tight capability claim, (ii) FMPy adds binary FMU handling and platform-specific
  shared libraries, which are new ways for a reviewer's install to fail, and
  installability is what JOSS actually grades, (iii) it would spend the schedule buffer
  that protects the writing start.
- **Assumptions**: a working demo on a branch with a README and a screen recording is
  credible for a job application without being in a published paper.
- **Open questions**: exact timing is under revision. A restructure was proposed —
  FMU export moved to Dec 2026–Jan 2027 gated on Tier-1 validation being on track by
  end of November, with the assimilation API and twin example pushed to post-JOSS
  (May–Aug 2027) — because the original Feb–April window is roughly triple capacity
  and lands after most of the Q1 application window. Not yet decided.
- **Standing constraint**: none of this may delay or compete with the core library work
  or the JOSS paper. If Tier-1 validation slips, Tier 1.5 slips with it. The paper wins
  every conflict.
- **Design commitment for (B)**: the observation operator is user-supplied. Accept an
  arbitrary `h(state) -> measurement` plus its Jacobian, or finite-difference it. If `h`
  is hardcoded to the demo's sensor layout, what shipped is a demo with an API-shaped
  wrapper rather than a capability.
- **Exclusions for (A)**: Co-Simulation only (no Model Exchange), FMI 2.0 only (no 3.0),
  no master algorithm or orchestration, no SSP or multi-FMU coupling, no C codegen or
  standalone binary FMUs, no variable-step negotiation, no general model import.
- **Exclusions for (B)**: no particle filters, no 4D-Var, no ensemble Kalman variants,
  no smoothing (filtering only), no automatic noise-covariance tuning.

## D-009 — Tier 1 exclusions, and the one conditional exception
2026-06-04

- **Decision**: excluded from Tier 1 — C++/CUDA/pybind11; multi-GPU or distributed
  anything; performance, speed, throughput, or scaling claims; hard BC enforcement;
  multiple sparse formats; FMI export and data assimilation; the LLM-design arc.
- **Conditional exception**: lightweight residual-based adaptive resampling is excluded
  as a *feature* but may be added as a **correctness contingency** if ENGD on the
  irregular 2D elasticity demo proves unstable without it.
- **Alternatives rejected**: shipping a fuller-featured library in Tier 1.
- **Why**: the JOSS bar is software quality, not feature count. Every additional feature
  is another surface to test, document, and keep installable, and an untested
  undocumented feature makes the submission weaker rather than stronger. The excluded
  items are deferred to Tier 2, not abandoned.
- **Assumptions**: reviewers will not read the exclusions as gaps if "future work" names
  them in one confident paragraph.
- **Open questions**: whether the adaptive-resampling contingency will be needed. If it
  is, log it as a correctness decision, not a performance one, and keep it minimal.

## D-008 — Soft BC enforcement only in Tier 1
2026-06-04

- **Decision**: implement soft (penalty-term) boundary condition enforcement only.
- **Alternatives rejected**: hard BC enforcement baked into the architecture, or both.
- **Why**: soft enforcement works for arbitrary geometry and is far simpler. Hard
  enforcement guarantees exact satisfaction and removes a sensitive weight, but touches
  the architecture and is only tractable for simple geometries — which conflicts with
  the irregular-geometry demo that justifies the graph representation.
- **Assumptions**: the loss-weight sensitivity that soft enforcement introduces is
  manageable on the chosen demos.
- **Open questions**: hard enforcement is a candidate Tier 2 feature.

## D-007 — Examples are cheap by design, with analytic or manufactured ground truth
2026-06-04

- **Decision**: every example must run on a laptop CPU in seconds and be verified
  against analytic or manufactured ground truth (Kirsch for plate-with-hole, modal
  solutions for beams, method of manufactured solutions elsewhere).
- **Alternatives rejected**: impressive large-scale or nonlinear transient examples.
- **Why**: for a JOSS tools paper the examples exist to show the tool works and is
  usable, not to solve a hard research problem. A reviewer runs the notebook and it
  should finish in seconds; an expensive example actively damages the usability claim
  that is the contribution. Expensive examples also risk failing for reasons unrelated
  to library correctness (PINNs smear discontinuities and drift on sharp transients),
  which would leave the paper's own demo undercutting its claim. Manufactured solutions
  additionally give exact ground truth for free, with no reference solver to build.
- **Assumptions**: cheap examples are sufficient to demonstrate the capability.
- **Open questions**: none. Hard nonlinear transient structural dynamics stays as the
  motivating story in the Statement of Need, never as an example.

## D-006 — Demo set: beam (validation) + 2D unstructured elasticity (keystone) + Cosserat (stretch)
2026-06-04

- **Decision**: (1) cantilever / flexible-link beam vibration, Euler-Bernoulli then
  Timoshenko if time allows — analytic modal solutions; (2) 2D linear elasticity on an
  unstructured triangular mesh, plate-with-hole (Kirsch) or L-shaped compliant-mechanism
  section — THE KEYSTONE; (3) static Cosserat-rod continuum arm — stretch goal only.
- **Alternatives rejected**: beam alone; large-deformation hyperelastic soft actuators;
  contact/impact problems.
- **Why**: the beam gives exact ground truth and a recognizable robotics framing, but it
  is 1D and a coordinate MLP would suffice, so it validates without positioning. The 2D
  unstructured-mesh demo is what justifies a graph representation over a coordinate MLP,
  and reviewers will probe that rationale directly — it does the most work in the paper.
  Contact/impact is excluded because nonsmooth solutions break the smoothness assumption
  PINNs rest on, so it would fail for reasons unrelated to the library.
- **Assumptions**: ENGD converges on the irregular 2D problem within the cheap-example
  budget.
- **Open questions**: if it does not, downscale the mesh or fall back to a manufactured
  solution before broadening scope. The Cosserat demo is nonlinear and lacks analytic
  ground truth, so it is explicitly optional.

## D-005 — Community is computational mechanics; robotics is the example domain
2026-06-06

- **Decision**: the underserved community claimed in the JOSS Statement of Need is
  computational mechanics — engineers solving PDEs on irregular unstructured meshes.
  Robotics elastodynamics (flexible manipulators, compliant mechanisms, continuum arms)
  is the example domain that makes demos concrete, not the community claim.
- **Alternatives rejected**: repositioning the Statement of Need onto robotics.
- **Why**: a mid-2026 survey found the robotics simulation community is well served by
  SOFA, MuJoCo, Isaac, Drake, and DiffTaichi, and predominantly wants fast,
  differentiable, real-time-capable surrogates for control rather than high-accuracy
  forward PDE solves. "Underserved" is therefore much harder to defend there than in
  mechanics, where accuracy genuinely matters. Using robotics only for examples keeps a
  real tooling gap while giving the demos a recognizable narrative.
- **Assumptions**: the mechanics framing remains defensible at submission.
- **Open questions**: the Statement of Need rests on two legs — the capability gap and
  the community — and needs a current feature survey as evidence for the first. That
  survey must be refreshed before submission, since the field moves quickly.

## D-004 — Optimizer interface is modular: curvature assembly separable from linear solve
2026-06-06

- **Decision**: expose curvature-matrix assembly and the linear solve as separable
  pieces rather than fusing them into one ENGD-specific step, so that KFAC, Dual NGD,
  and ANaGRAM-style variants can drop in behind the same interface.
- **Alternatives rejected**: a single monolithic ENGD step.
- **Why**: a mid-2026 literature check found KFAC-for-PINNs (2024), ANaGRAM (ICLR 2025),
  Dual NGD (2025), and Woodbury/SPRING ENGD (NeurIPS 2025) all active. ENGD is not
  superseded, but by submission a reviewer may reasonably ask why ENGD specifically. The
  defensible framing is that the library implements *function-space second-order
  optimization* with ENGD as its reference implementation, and that framing is only
  credible if the interface actually admits alternatives.
- **Assumptions**: a general interface can be designed without knowing the details of
  every variant.
- **Open questions**: whether the abstraction survives contact with a second
  implementation. It will not be tested until a variant is actually added.

## D-003 — Second-order optimizer: ENGD, not KFAC
2026-05-30

- **Decision**: ship Energy Natural Gradient Descent (Müller & Zeinhofer 2023) as
  Tier 1's distinguishing optimizer.
- **Alternatives rejected**: KFAC for PINNs (Dangel et al. 2024).
- **Why**: validation transparency. ENGD's correctness signal is legible — the error
  falls toward machine precision or it does not — whereas a wrong KFAC factorization
  still runs, still trains, and still produces a plausible loss curve while computing
  the wrong preconditioner. That matters disproportionately when implementing from a
  paper, solo, on a deadline, with an assistant that can write the code but cannot
  verify its numerics. ENGD's orders-of-magnitude accuracy result is also Tier 1's
  contribution, whereas KFAC's pitch is scaling, which belongs to a later tier. ENGD
  additionally avoids a Taylor-mode AD prerequisite, shortening the critical path.
- **Tie-breaker reasoning**: prefer a visible performance problem (ENGD too slow per
  step, which is profilable) over an invisible correctness problem (KFAC subtly wrong,
  possibly undetected).
- **Assumptions**: ENGD converges acceptably on the chosen demos within the cheap-example
  budget.
- **Open questions**: KFAC is deferred to a later optimizer arc, layered on the
  distributed engine where its scaling advantage is actually meaningful.

## D-002 — Tier 1 is pure PyTorch, not C++/CUDA
2026-06-06

- **Decision**: Tier 1 contains no C++, CUDA, or pybind11, and nothing may depend on a
  GPU. The compiled backend moves entirely to Tier 2.
- **Alternatives rejected**: writing the Tier 1 library in C++ from the start to avoid
  a later rewrite.
- **Why**: C++ buys nothing for a JOSS submission, because Tier 1 makes no performance
  claim — it would pay the full cost of the hardest engineering for a benefit the
  Statement of Need does not invoke. It also damages the installability that IS the
  contribution: a compiled library needs a toolchain and platform-specific builds,
  reproducing exactly the accessibility problem being positioned against. And it would
  put the graduation-critical deliverable on the riskiest path while blowing the
  schedule.
- **On the rewrite concern**: the PyTorch library is not throwaway. It is the
  correctness oracle against which the Tier-2 compiled backend will be validated —
  same inputs, same update, do the numbers match — and validating a fast kernel without
  a trusted reference is the hardest part of that work. It is also the JOSS artifact and
  the thesis, with its own life.
- **Assumptions**: designing Tier 1 interfaces with an eventual compiled backend in mind
  costs little now and eases the Tier 2 transition.
- **Open questions**: whether message passing uses PyTorch Geometric or hand-rolled
  scatter primitives. Not yet decided; to be logged separately with rationale.

## D-001 — Two-tier structure: JOSS tools paper first, SIAM performance paper later
2026-06-06

- **Decision**: Tier 1 is a correct, usable, well-tested PyTorch PI-GNN library with
  ENGD, yielding one body of work and two documents — the Master's thesis and a JOSS
  tools paper. Tier 2 is the C++/CUDA and multi-GPU engine, yielding a SIAM performance
  paper. Tier 1 must be finished and submitted before Tier 2 starts.
- **Alternatives rejected**: a single SIAM paper covering both; making the first paper a
  performance contribution; folding multi-GPU into the first paper.
- **Why**: the full scoped library does not fit the available time, and building it as a
  monolith before benchmarking would consume the window in which the performance thesis
  could still be checked. Splitting puts the graduation-critical deliverable on the
  low-risk path (correctness and usability, no performance claim, no compiled code) and
  quarantines the hard, failure-prone HPC work in a tier where nothing is at stake
  because the degree is already secured and a publication banked. JOSS is the right
  venue for Tier 1 because it reviews software quality rather than novelty, which is
  exactly what Tier 1 offers, and JOSS explicitly coexists with a later research paper
  about the same software.
- **Assumptions**: JOSS accepts the library on software-quality grounds; the two papers
  are distinct enough contributions to avoid dual-publication conflict (multi-GPU is
  substantially new content beyond single-GPU).
- **Open questions**: confirm dual-publication norms with the supervisor or a
  math-department contact before submitting Tier 1.
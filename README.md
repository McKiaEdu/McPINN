# McPINN

A physics-informed graph neural network library with second-order (ENGD) optimization
for partial differential equations on irregular meshes.

> Master's research project, Computational Science and Engineering, McMaster University.

## Project status

Early development. The package is not yet installable, no components have been
implemented, and the directory tree currently holds placeholders. This document
describes the intended scope and grows as the library is built. An OSI-approved
license will be added before the first release.

## Statement of need

A physics-informed neural network (PINN) solves a partial differential equation (PDE)
by embedding the equation residual, the amount by which a candidate solution fails to
satisfy the equation, directly in the training loss. The usual construction takes
spatial coordinates as input to a multilayer perceptron (MLP), which suits problems
posed on coordinate grids. Many problems in computational mechanics are instead posed
on irregular unstructured meshes, where a graph representation with message passing
fits the discretization more naturally. Networks of this second kind are
physics-informed graph neural networks (PI-GNNs).

Two lines of work have developed separately. Function-space second-order optimizers,
of which Energy Natural Gradient Descent (ENGD, Müller and Zeinhofer 2023) is the
reference method, drive the solution error of a PINN toward machine precision, and
every published method of this family is applied to coordinate-input MLPs. Physics-informed
graph neural networks (Gao et al. 2022 and successors) are trained with Adam or L-BFGS.
A survey of the literature and of the established libraries (DeepXDE, PhysicsNeMo, PINA,
jinns, TorchPhysics) conducted in mid-2026 found the intersection of the two unoccupied.

McPINN provides that intersection: function-space second-order optimization for
physics-informed graph neural networks, with ENGD as its reference implementation.

The intended users are computational mechanics practitioners who solve PDEs on
irregular unstructured meshes. The worked examples are drawn from robotics
elastodynamics (flexible manipulators, compliant mechanisms, continuum arms), a domain
chosen to keep the examples concrete rather than to serve robotics simulation, which is
already covered by established tools such as SOFA, MuJoCo, and Isaac.

## Scope

McPINN is a pure-PyTorch library. Its contribution is capability and usability, and it
also serves as the correctness reference against which a later compiled backend is
checked, so reproducible and verifiable numerical results are the priority.

Planned components:

- Mesh handling and construction of graph representations from unstructured meshes.
- Message-passing architectures for physics-informed graph neural networks.
- Differential operators and PDE residual assembly.
- An optimizer interface that separates curvature-matrix assembly from the linear
  solve, with ENGD as the reference implementation and Adam and L-BFGS as baselines.
- PDE definitions with manufactured and analytic solutions, so correctness is checked
  against exact ground truth rather than read from a plot.

Deliberately out of scope for this release: C++, CUDA, and pybind11 backends;
multi-GPU and distributed execution; hard boundary-condition enforcement. A local GPU
may be used for convenience, and no part of the library depends on one. Every shipped
example runs on a laptop CPU in seconds.

## Roadmap

The build order changes one element at a time, so that a failure localizes to the
element that changed.

1. One-dimensional Poisson equation with a manufactured solution, plain MLP, Adam.
   This establishes the problem, residual, and training plumbing against analytic
   ground truth.
2. Replace the MLP with message passing on a one-dimensional graph. The computed
   answer stays the same, which isolates the graph machinery against a trusted result.
3. Replace Adam with ENGD. The solution error is expected to fall toward machine
   precision. This is the central validation step of the project.
4. Cantilever beam vibration (Euler-Bernoulli, then Timoshenko if time allows), where
   closed-form modal solutions and natural frequencies give exact ground truth.
5. Two-dimensional linear elasticity on an unstructured triangular mesh, either a plate
   with a hole (Kirsch solution) or an L-shaped compliant-mechanism section. This is the
   example that motivates a graph representation over a coordinate MLP.
6. Extract the shared abstractions once three uses exist, then refactor into
   configurable classes.

## Installation

Not yet available. Installation instructions, the supported Python version, and a
pinned dependency file will be added once the package is installable.

## Reproducing the results

Not yet available. This section will give the commands that regenerate every reported
figure and number from a clean checkout.

## Citation and references

Bibliographic entries for the works named above are collected in the accompanying
paper, in `paper/`.

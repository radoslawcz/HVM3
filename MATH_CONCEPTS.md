# Mathematical Concepts behind HVM

This document distills the core mathematical ideas that underpin the Higher-order Virtual Machine (HVM). The primary theoretical foundation is the **Interaction Calculus (IC)**, a novel model of computation.

## The Interaction Calculus (IC)

The Interaction Calculus is the core mathematical model behind HVM. It is a term-rewriting system, much like the more familiar Lambda Calculus (λC), which provides a foundation for functional programming. However, IC introduces a different set of primitive operations that lead to fundamentally different computational behaviors.

The basic components (terms) of the Interaction Calculus are:
- **Variables (`VAR`)**: Placeholders for terms, similar to variables in algebra or λC.
- **Erasure (`ERA`)**: Represented as `*`, this term explicitly discards a value.
- **Lambdas (`LAM`)**: `λx.t`, representing a function abstraction, similar to λC.
- **Applications (`APP`)**: `(f a)`, representing the application of a function `f` to an argument `a`.
- **Superposition (`SUP`)**: `&L{a,b}`, a primitive that can be thought of as a pair of two terms, `a` and `b`, existing in the same location. The label `L` is a numeric value that affects interactions.
- **Duplication (`DUP`)**: `!&L{x,y}=a;b`, a primitive that "projects" a value `a` into two separate locations, `x` and `y`, which can then be used in the body `b`. It is the computational dual of superposition.

These elements and their interactions form the basis of the HVM's execution model.

## Key Differences from Lambda Calculus

While IC shares concepts like lambdas and applications with λC, it diverges in several crucial ways that grant it unique properties:

1.  **Affinity (Linearity)**: In IC, variables are *affine*, meaning they can be used at most once. This is a concept borrowed from **Linear Logic**. If a variable is not used, it must be explicitly discarded with an `ERA` term. This constraint makes the system resource-aware and simplifies garbage collection, as there are no complex reference counting or tracing algorithms needed.

2.  **Global Scopes**: Unlike the lexical scoping of λC, variables in IC have global scope. A binder `λx` can be referenced from anywhere in the program. This seems counter-intuitive at first but is managed by a global substitution map. When a lambda is applied, like `(λx.f a)`, the system simply registers that `x` is now substituted by `a` globally. This avoids the costly process of traversing a term to perform substitutions and the complexities of handling name capture (variable shadowing).

3.  **First-Class Superposition and Duplication**: `SUP` and `DUP` are not features that can be encoded; they are primitive to the calculus.
    *   `SUP` allows two different terms to be "superposed" in a single location.
    *   `DUP` allows a term to be copied and used in two different places.
    These primitives are the key to unlocking one of IC's most powerful features: optimal reduction.

## Optimal Reduction

One of the most significant consequences of the IC's design is its ability to perform **optimal reduction**. In the context of term rewriting, "optimal" means that every computational step is shared to the maximum extent possible. No computation is ever duplicated, and no computation is ever performed unnecessarily.

This is achieved primarily through the interaction between `DUP` and `SUP` nodes. Consider a typical lambda calculus expression like `(λx.(x+x) (2*3))`. A naive evaluator would first compute `2*3` to get `6`, then substitute it into the body, resulting in `6+6`. The multiplication was performed once. However, in an expression like `(λf.(f 1)+(f 1)) (λy.y*y)`, the function `λy.y*y` is passed to the outer lambda. A naive evaluator would compute `(1*1) + (1*1)`, performing the multiplication twice.

The Interaction Calculus avoids this duplication. When a term that is needed in multiple places (via a `DUP` node) is evaluated, it can be reduced "under the binders". The `DUP-SUP` interaction rules are designed to push the duplication inwards, past other operations, until it reaches the values themselves. This effectively means that computations are performed only once, and the *results* are shared.

This property can lead to exponential speedups for certain classes of programs compared to traditional evaluation strategies for the Lambda Calculus. It realizes a long-standing goal of computer science, dating back to research on optimal graph-reduction algorithms for the lambda calculus, but in a much simpler and more direct formalism. The HVM, as an implementation of IC, brings this theoretical optimality into practice.

## Theoretical Underpinnings

The ideas in the Interaction Calculus are not entirely new; they are built upon a rich history of theoretical computer science:

-   **Linear Logic**: Introduced by Jean-Yves Girard, Linear Logic is a "resource-sensitive" logic. Unlike classical logic where a premise can be used as many times as needed, in linear logic, premises must be used exactly once. The affinity of variables in IC is a direct application of this principle. The `DUP` primitive in IC corresponds to the "of course" modality `!` in linear logic, which allows a resource to be duplicated.

-   **Interaction Combinators**: Developed by Yves Lafont, this is a graphical model of computation based on a very simple set of interaction rules between agents. The Interaction Calculus can be seen as a textual representation of Lafont's Interaction Combinators, providing a more conventional syntax for these ideas. The core `DUP-SUP` and `APP-LAM` interactions in IC are direct analogues of the annihilation and commutation rules in Lafont's system.

By building on these foundations, the Interaction Calculus provides a robust and powerful model of computation that is both theoretically elegant and practically efficient, as demonstrated by the HVM.
# Signs of multipliers, costate and duals

One sign convention for every multiplier of the ecosystem: the multipliers of a nonlinear
program returned by any solver, the costate and the multipliers of the maximum principle,
and the duals stored in the solution of an optimal control problem. A multiplier belongs to
a **constraint** and to a **problem**, not to the way they are written, nor to the solver or
the method that computes it.

Identifiers: `SGN1`–`SGN6` (see [`README.md`](../README.md#rule-identifiers)).

Notation. A minimisation `min F(x)` or a maximisation `max F(x)`, with constraint rows
`l ≤ c(x) ≤ u` and bounds `lvar ≤ x ≤ uvar`. `F_min` is the objective of the equivalent
minimisation: `F_min = F` for `min F`, `F_min = -F` for `max F`. On a row, `μL, μU ≥ 0`
are the multipliers of the lower and upper side; on the bounds, `zL, zU ≥ 0`.

---

## SGN1 — `min f` and `max -f` have the same multipliers

**Rule.** The multipliers of a maximisation `max F` are those of the equivalent
minimisation `min -F`. They do not change sign when a problem is written `max -f` instead of
`min f`; only the objective value does.

**Why.** `min f` and `max -f` are the same problem: same solution, same active constraints,
same sensitivity of the optimal *point* to the constraints. NLP solvers disagree on the
other choice (some return the multipliers of the objective as written, so that every
multiplier changes sign with the sense; some flip the constraint rows but not the bounds),
so without a rule the duals of a problem depend on the solver and on the sense of the
criterion.

**Check.** `exact` (to be tooled): a test solves the same problem as `min f` and as
`max -f` and compares every multiplier (see [`SGN6`](#sgn6--normalised-multipliers-are-tested-in-two-layers)).

---

## SGN2 — An inequality has a nonnegative multiplier, however it is written

**Rule.** Every inequality multiplier is **nonnegative**, for a minimisation and for a
maximisation: one multiplier per active side, `μL ≥ 0` for `c(x) ≥ l`, `μU ≥ 0` for
`c(x) ≤ u`, `zL, zU ≥ 0` for the bounds of the variables. The dual of a constraint with a
**single** finite bound is the multiplier of that side, so it is the same, and `≥ 0`, for
every way of writing the constraint:

| Written as | Dual |
| --- | --- |
| `g(x) ≤ 0` | `μ ≥ 0` |
| `-g(x) ≥ 0` | the same `μ` |
| `x₁ ≤ a` as a bound of the variable | the same `μ` |
| `-x₁ ≥ -a` as a constraint row | the same `μ` |

**Why.** These are the same constraint. A signed multiplier per row (`y = μU − μL`, the
raw output of most solvers) gives `+μ` for `g ≤ 0` and `−μ` for `-g ≥ 0`, so the sign of a
dual would tell how the user happened to write the constraint, not anything about the
problem.

**Check.** `exact` (to be tooled), by the tests of
[`SGN6`](#sgn6--normalised-multipliers-are-tested-in-two-layers).

---

## SGN3 — Equalities and two-sided constraints carry a signed multiplier

**Rule.** An **equality** `c(x) = b`, and a constraint with **two** finite bounds
`l ≤ c(x) ≤ u` (`l < u`), have one signed multiplier `y = μU − μL`, given by the
stationarity of the Lagrangian of the equivalent minimisation:

```math
\nabla F_{\min}(x) + J(x)^\top y - z_L + z_U = 0, \qquad z_L, z_U \ge 0.
```

For a two-sided constraint, `y ≥ 0` when the upper side is active and `y ≤ 0` when the lower
side is (at most one side is active). For a two-sided bound of a variable, the signed dual is
`zU − zL`. An equality has no preferred side: `c(x) = 0` and `-c(x) = 0` have opposite
multipliers.

**Why.** An equality has no sign condition, so its multiplier keeps the sign of the
stationarity above, which is also the sign of the costate of the dynamics (an equality).
For a two-sided constraint, the signed value says which side is active, with a single number
per constraint.

**Check.** `judgment`.

---

## SGN4 — The pseudo-Hamiltonian is always maximised

**Rule.** For the maximum principle, the pseudo-Hamiltonian of a problem with criterion
`∫ℓ` (and dynamics `f`, path constraints `g ≥ 0`) is

```math
H(t,x,p,u,v) = p \cdot f(t,x,u,v) + p^0\,\ell(t,x,u,v) + \mu \cdot g(t,x,u,v),
\qquad g \ge 0, \quad \mu \ge 0,
```

and it is **maximised** in `u`, for a minimisation and for a maximisation. Only `p⁰` depends
on the sense: `p⁰ ≤ 0` for a minimisation (`p⁰ = −1` in the normal case), `p⁰ ≥ 0` for a
maximisation (`p⁰ = +1`). A constraint is adjoined normalised as `g ≥ 0` on its active side;
its dual, in the solution, is `μ` for a constraint with a single bound
([`SGN2`](#sgn2--an-inequality-has-a-nonnegative-multiplier-however-it-is-written)), `−μ`
or `+μ` on the lower or upper side of an equality or a two-sided constraint
([`SGN3`](#sgn3--equalities-and-two-sided-constraints-carry-a-signed-multiplier)).

**Why.** `p⁰ℓ` is the same for `min ∫ℓ` (`p⁰ = −1`) and for `max ∫(−ℓ)` (`p⁰ = +1`), so the
control, the costate and the multipliers are the same, as [`SGN1`](#sgn1--min-f-and-max--f-have-the-same-multipliers)
requires for the direct method. The other choice (`p⁰ = −1` always, `H` minimised for a
maximisation) makes the costate and the multipliers change sign with the sense.

**Check.** `exact` (to be tooled): a test computes the same extremal for `min ∫ℓ` and
`max ∫(−ℓ)` and compares the costate and the multipliers.

---

## SGN5 — The same constraint has the same dual, for every kind and every method

**Rule.** A constraint has the same dual (same sign) whether it is declared as a path
constraint or as a box, and whether the problem is solved by a direct method (from the
multipliers of the NLP) or by an indirect one (from the multipliers of the maximum
principle). The duals of a solution are read by the label of the constraint.

**Why.** The dual is a property of the constraint and of the problem
([`SGN1`](#sgn1--min-f-and-max--f-have-the-same-multipliers)–[`SGN4`](#sgn4--the-pseudo-hamiltonian-is-always-maximised)).
Comparing a direct and an indirect solution, or a constraint written as a box and as a path
constraint, is the usual way to check a solution; it is only possible with one convention.

**Exceptions.** The **scale** of a time-dependent dual (a density, divided by the quadrature
weight, or the raw multiplier of the discretised problem) is not fixed by this rule.

**Check.** `exact` (to be tooled): a cross-package test compares the duals of a direct and
of an indirect solve of a problem with a closed form.

---

## SGN6 — Normalised multipliers are tested in two layers

**Rule.** A package that wraps NLP solvers brings their multipliers to this convention
before returning them, and tests it in two layers, on a problem with closed-form multipliers
written in every way the rules above cover (`min f` / `max -f`, `g ≤ 0` / `-g ≥ 0`, bound /
row, active lower and upper sides, an equality, a fixed variable):

1. the **native** convention of each solver, on its raw output, with the solver and its
   version in the name of the test;
2. the **normalised** output, which must be the same for every way of writing the problem
   and for every solver.

The raw output stays available (an option or a lower-level call).

**Why.** Solvers change their conventions upstream. With the two layers, a failure says
where the change is: (1) fails, a solver moved and its normalisation must follow; (1)
passes and (2) fails, the normalisation is wrong. A single test of the normalised output
cannot tell the two apart.

**Check.** `judgment`: the two layers exist and cover the cases above.

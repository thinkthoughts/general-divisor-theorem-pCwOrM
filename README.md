# General Divisor Theorem (GDT): Lean 4 Formalization

[![Lean 4](https://img.shields.io/badge/Lean_4-v4.21.0-blue.svg)](https://github.com/leanprover/lean4)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23316070.svg)](https://doi.org/10.5281/zenodo.23316070)
[![License](https://img.shields.io/badge/License-Apache_2.0-green.svg)](LICENSE)
[![Kernel Audit](https://img.shields.io/badge/Kernel_Audit-no_project_axioms-brightgreen.svg)](#kernel-dependency-audit)

![General Divisor Theorem Lean 4 Infographic](gdt_twitter_infographic.png)

Lean 4 formalization of the **General Divisor Theorem (GDT)**: exact density correction under residue conditioning.

The mathematical specification originates with **Dan Hawkley / thinkthoughts**
([@Precedent_Vice](https://x.com/Precedent_Vice)):

> *The General Divisor Theorem: Exact Density Correction under Residue Conditioning* (Draft v3.3)  
> Paper: [https://antiviolence.ai/gdt.pdf](https://antiviolence.ai/gdt.pdf)  
> Local mirror: [`docs/gdt.pdf`](docs/gdt.pdf)  
> Original challenge repository: [thinkthoughts/general-divisor-theorem](https://github.com/thinkthoughts/general-divisor-theorem)

Zenodo archive: [10.5281/zenodo.23316070](https://doi.org/10.5281/zenodo.23316070)

---

## Kernel-verified milestone — October 2026

The current `gdt-kernel-bridge` development extends the formalization through:

- admissible / inadmissible dichotomy;
- exact periodicity;
- exact `φ(R)` count per period;
- exact arbitrary-bound counting;
- natural-density convergence;
- the density-correction identity;
- a unified top-level theorem.

The current project state has:

- a successful `lake build`;
- no project `axiom` declarations in the Lean sources;
- no Lean `sorry` terms in the Lean sources;
- explicit `#print axioms` checks on the headline theorems.

The audited headline theorems report only:

```text
[propext, Classical.choice, Quot.sound]
```

No `sorryAx` or project-defined mathematical axiom appears in those dependency reports.

---

## 1. Mathematical specification

Let \(N,m \in \mathbb N^+\) and \(a \in \mathbb Z\). Define

\[
d=\operatorname{rad}(\gcd(m,N)),\qquad
R=\frac{\operatorname{rad}(N)}{d},\qquad
T_{\min}=mR.
\]

The predicate

```lean
S_indicator N m a n
```

means

\[
n\equiv a \pmod m
\quad\text{and}\quad
\gcd(n,N)=1.
\]

### Inadmissible branch

If

\[
\gcd(|a|,d)>1,
\]

then there are no accepted integers in the prescribed residue class.

### Admissible branch

If

\[
\gcd(|a|,d)=1,
\]

the development proves periodicity with

\[
T_{\min}=mR=\operatorname{lcm}(m,\operatorname{rad}(N)),
\]

together with exactly

\[
\varphi(R)
\]

accepted values in one complete period.

---

## 2. Exact counting

`DensityBridge.lean` proves the exact one-period result:

```lean
theorem gdt_count_per_period ...
```

with conclusion

\[
\operatorname{countOnePeriod}(N,m,a)=\varphi(R).
\]

`DensityCount.lean` lifts that result to arbitrary bounds. For every `q s : Nat`,

\[
C(qT_{\min}+s)=q\,\varphi(R)+C(s),
\]

where `C(X)` is `countBelow N m a X`.

In particular,

\[
C(qT_{\min})=q\,\varphi(R).
\]

This is an exact counting identity, rather than an asymptotic approximation.

---

## 3. Natural density

`DensityLimit.lean` uses the canonical decomposition

\[
X=\left\lfloor\frac{X}{T_{\min}}\right\rfloor T_{\min}
  +(X\bmod T_{\min})
\]

together with the exact counting formula and a uniformly bounded remainder.

The theorem

```lean
GeneralDivisorTheorem.density_limit
```

proves over `ℝ` that

\[
\frac{C(X)}{X}\longrightarrow
\frac{\varphi(R)}{T_{\min}}
=
\frac{\varphi(R)}{mR}.
\]

---

## 4. Density correction

The naive independent-density expression is

\[
\frac{\varphi(N)}{Nm}.
\]

`CorrectionBridge.lean` proves the integer identity

\[
\varphi(R)\,N\,\varphi(d)
=
dR\,\varphi(N).
\]

Hence the exact admissible density differs from the naive expression by

\[
\frac{d}{\varphi(d)}.
\]

Equivalently,

\[
\frac{\varphi(R)}{mR}
=
\frac{d}{\varphi(d)}
\frac{\varphi(N)}{Nm}.
\]

The correction depends on the shared prime support of `m` and `N`, rather than on the admissible residue `a`.

---

## 5. Unified theorem

`GDTMain.lean` packages the current proof chain as

```lean
GeneralDivisorTheorem.general_divisor_theorem
```

for positive `N` and `m`.

The theorem splits into the exact arithmetic dichotomy:

### Inadmissible

```text
∀ n, ¬ S_indicator N m a n
```

or:

### Admissible

the conjunction of

```text
gcd(|a|,d) = 1
periodicity by T_min
exact φ(R) count per period
exact q*T_min+s counting law
natural-density limit φ(R)/T_min
integer correction identity
```

---

## 6. Lean module structure

```text
GeneralDivisorTheorem.lean
    │
    ├── foundational definitions
    ├── admissibility / inadmissibility
    ├── radical factorization
    ├── period and lcm results
    ├── exponent blindness
    └── specialization theorems
             │
             ▼
DensityBridge.lean
    │
    └── exact φ(R) count in one period
             │
             ▼
DensityCount.lean
    │
    ├── translated-period counts
    ├── q-period count
    └── exact q*T+s count
             │
             ▼
DensityLimit.lean
    │
    ├── quotient/remainder count
    ├── bounded remainder
    ├── real-valued error bound
    └── natural-density convergence
             │
             ├──────────────┐
             ▼              ▼
CorrectionBridge.lean   CoprimeBridge.lean
    │                       │
    └── correction identity └── compatibility bridge
             │
             ▼
GDTMain.lean
    │
    └── general_divisor_theorem
```

`VolkanBridge.lean` contains the repository's compatibility layer between the alternate representations used during development.

---

## 7. Specializations

The core development also includes the GDT specializations for cases including:

- `m ∣ N`;
- unitary divisors;
- squarefree / primorial cases.

It also contains distinguishing examples designed to separate the general theorem from stronger naive admissibility conditions.

---

## 8. Building

The repository is pinned to Lean 4.21.0 / Mathlib v4.21.0.

Build the project with:

```bash
lake build
```

Individual modules may also be built directly, including:

```bash
lake build GeneralDivisorTheorem
lake build DensityBridge
lake build DensityCount
lake build DensityLimit
lake build CorrectionBridge
lake build CoprimeBridge
lake build GDTMain
lake build VolkanBridge
```

---

## 9. Source audit

A source-level audit can be run with:

```bash
grep -R -nE '\b(axiom|sorry)\b' \
  --include='*.lean' \
  --exclude-dir='.lake' .
```

At the current verified reading point there are no actual project `axiom` declarations or Lean `sorry` terms.

Comments containing either word should not be confused with Lean declarations or proof holes.

---

## 10. Kernel dependency audit

`Audit.lean` contains:

```lean
#print axioms GeneralDivisorTheorem.general_divisor_theorem
#print axioms GeneralDivisorTheorem.density_limit
#print axioms GeneralDivisorTheorem.gdt_correction_identity
#print axioms GeneralDivisorTheorem.gdt_count_per_period
#print axioms GeneralDivisorTheorem.coprime_m_R
#print axioms GeneralDivisorTheorem.coprime_m_R_bridge
```

Run:

```bash
lake env lean Audit.lean
```

The current result for each audited theorem is:

```text
[propext, Classical.choice, Quot.sound]
```

These are standard Lean foundations used through Mathlib. The report contains no `sorryAx` and no project-defined GDT axiom.

---

## 11. Computational checks

The repository also contains executable arithmetic checks:

```bash
python scripts/verify_gdt.py
python scripts/verify_bridge.py
```

These provide independent computational checks and are complementary to, rather than substitutes for, the Lean kernel proofs.

---

## 12. Citation and attribution

```bibtex
@misc{dagli2026gdt_lean4,
  author    = {Da\u{g}l\i, Volkan and Da\u{g}l\i, Zerrin and Da\u{g}l\i, Da\u{g}han},
  title     = {Machine-Certified Lean 4 Formalization of the General Divisor Theorem: Exact Density Correction under Residue Conditioning},
  year      = {2026},
  month     = {October},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.23316070},
  url       = {https://doi.org/10.5281/zenodo.23316070}
}

@misc{thinkthoughts2026gdt,
  author       = {Dan Hawkley},
  title        = {The General Divisor Theorem: Exact Density Correction under Residue Conditioning},
  year         = {2026},
  howpublished = {\url{https://antiviolence.ai/gdt.pdf}},
  note         = {Draft v3.3}
}
```

The mathematical GDT specification and original Lean challenge were developed by Dan Hawkley / `thinkthoughts`. This repository contains the subsequent Lean 4 implementation, collaborative bridge work, verification infrastructure, and the `gdt-kernel-bridge` development extending the formalization through exact counting, density, correction, and unified theorem packaging.

For archived repository metadata and its author list, see the Zenodo record above.

---

## 13. Acknowledgments

Computing and proof-verification infrastructure acknowledgments from the archived project remain available in the Zenodo record and repository history.

---

## 14. License

Apache-2.0.

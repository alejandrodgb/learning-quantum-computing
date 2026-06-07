# Quantum Computing — Learning Project

A self-directed path into quantum computing. It runs as **two forks in lockstep**: a *theory* track that builds intuition and a *project* track that proves each idea in code. 

---

## Getting started

The environment is defined in [`requirements.yml`](./requirements.yml) (a conda environment file).

```bash
# create the environment (note: -f is required since the file isn't named environment.yml)
conda env create -f requirements.yml

# activate it
conda activate quantum

# launch notebooks
jupyter lab
```

Notes:
- The Qiskit ecosystem packages install via `pip` inside the conda env; the scientific stack (numpy/scipy/matplotlib/pandas/jupyterlab) comes from conda-forge.
- Version floors in `requirements.yml` were valid in early 2026 and are **not** guaranteed latest.
- `pyscf` (Phase 4) can be finicky on Apple Silicon; if the pip install fails, move it into the conda `dependencies:` block.

---

## Repository structure

```
.
├── README.md                          # this file
├── requirements.yml                   # conda environment
├── .gitignore
└── projects/
    ├── 00_state_vector_simulator/     # Project 0  ·  Phase 0
    │   └── qsim.py
    ├── 01_gates_and_bloch/            # Project 1  ·  Phase 1
    ├── 02_bell_chsh/                  # Project 2  ·  Phase 2
    ├── 03_grover/                     # Project 3  ·  Phase 3
    ├── 04_vqe_h2/                     # Project 4  ·  Phase 4
    └── 05_qaoa_portfolio/             # Project 5  ·  Phase 5 (capstone)
```

Each project folder holds its notebook(s) and a short README noting what the result taught you.

---

## How the two forks connect

| Phase | THEORY (make it intuitive) | PROJECT (prove you get it) |
|-------|---------------------------|----------------------------|
| 0 | Amplitudes & interference — the real mental shift | Build a 1-qubit state-vector simulator in NumPy |
| 1 | The single qubit & the Bloch sphere | Extend the simulator to gates; mirror it in Qiskit |
| 2 | Multiple qubits & entanglement | Bell state + CHSH inequality test |
| 3 | Interference as the engine of speedup | Implement Grover's search; watch amplitudes move |
| 4 | The NISQ reality: hybrid variational circuits | VQE for the ground-state energy of H₂ |
| 5 | Applications: optimization | QAOA portfolio optimizer |

Work the columns in lockstep — finish the theory of a phase, then ship its project before moving on.

---

## The one idea to internalize first

Most newcomers carry a wrong intuition: "a quantum computer tries all answers at once." That's misleading and will block you for months.

The accurate picture: a quantum state is a vector of **complex amplitudes**, one per possible outcome. You never read amplitudes directly — measurement gives you an outcome with probability equal to the *squared magnitude* of its amplitude. A quantum algorithm is a choreography that makes the amplitudes of **wrong answers cancel** (destructive interference) and **right answers reinforce** (constructive interference), so that when you finally measure, the right answer is overwhelmingly likely.

Speedup comes from interference, not from parallelism. Hold onto that and the rest gets much easier.

---

## Spine resources (these carry most of the plan)

Rather than a different resource per phase, three free "backbone" materials each span several phases. Anchor on these; the per-phase lists below are mostly pointers into them.

- **Watrous — "Understanding Quantum Information and Computation"** (IBM Quantum Learning). Free, university-level, recently completed: 16 video lessons plus textbook text by John Watrous (IBM Quantum's technical director for education). Deliberately emphasizes timeless concepts over fast-changing details, so it won't go stale. Its course units map almost one-to-one onto Phases 1–3.
  - Hub: https://quantum.cloud.ibm.com/learning/en
  - Background / full-release note: https://www.ibm.com/quantum/blog/understanding-quantum-information-and-computation
- **Quantum Country — "Quantum computing for the very curious"** (Matuschak & Nielsen). Free primer that assumes only basic linear algebra + complex numbers. Its trick is a "mnemonic medium": embedded review questions plus spaced-repetition email prompts to fight forgetting. Best *intuition* builder for Phases 0–3.
  - https://quantum.country/qcvc  (essay series index: https://quantum.country/)
- **Nielsen & Chuang, *Quantum Computation and Quantum Information*** (Cambridge University Press). The standard reference — keep it for the rigorous lookup when the two above gloss something. A book to buy or borrow, not a free download. Publisher page: https://www.cambridge.org/9781107002173

**Ongoing / community:** Qiskit Global Summer School 2026 is free with a beginners-only track — https://www.ibm.com/quantum/blog/qiskit-summer-school-2026  `[unverified]` (confirm the dates yourself).

---

# FORK 1 — THEORY (make quantum intuitive)

### Phase 0 — Amplitudes & interference
**Goal:** stop thinking in bits, start thinking in amplitude vectors.
- A 1-qubit state is `α|0⟩ + β|1⟩` with `α, β` complex and `|α|² + |β|² = 1`.
- Measurement → outcome `0` with prob `|α|²`. The phase of `α`, `β` is invisible to a single measurement but drives interference later.
- Why interference matters: a real number can be negative; amplitudes can be negative or complex, so two paths to the same outcome can cancel. Classical probabilities only add.

**You understand it when:** you can explain why `H` applied twice returns `|0⟩` to `|0⟩` (the two paths interfere) without invoking "parallel universes."

**Materials:**
- Quantum Country, *Quantum computing for the very curious* — Part I (the state of a qubit): https://quantum.country/qcvc
- Scott Aaronson, *Intro to Quantum Information Science* lecture notes — free for personal/academic use; best for the "why is this strange" conceptual layer: https://scottaaronson.blog/?p=3943
- *Optional intuition:* 3Blue1Brown — use *Essence of Linear Algebra* if vector-space mechanics feel rusty. Note: he has no dedicated QC course, so this is supplementary only. https://www.3blue1brown.com/

### Phase 1 — The single qubit & the Bloch sphere
**Goal:** see one qubit geometrically.
- The Bloch sphere: any pure 1-qubit state is a point on a sphere. Gates are rotations of that sphere.
- Core gates: `X` (bit flip), `Z` (phase flip), `H` (the superposition-maker), and the rotation gates `Rx, Ry, Rz`.
- Global vs relative phase: global phase is unobservable; relative phase is everything.

**You understand it when:** given a gate, you can say roughly where it moves a point on the sphere, and you can explain why `H` sits a state on the equator.

**Materials:**
- Watrous, *Basics of quantum information* (states, measurements, circuits): https://quantum.cloud.ibm.com/learning/en
- Quantum Country, *Quantum computing for the very curious* — Part II (quantum logic gates): https://quantum.country/qcvc
- Interactive Bloch sphere (drag a state, watch probabilities across bases) — spend real time here:
  - https://blochsphereviz.com/
  - https://www.jonvet.com/tools/quantum-state-simulator

### Phase 2 — Multiple qubits & entanglement
**Goal:** understand where the exponential state space comes from, and what entanglement actually is.
- `n` qubits → a `2ⁿ`-dimensional amplitude vector, built by tensor product.
- Entanglement: a 2-qubit state that **cannot** be written as a product of two single-qubit states. Measuring one instantly constrains the other, with correlations no classical local model reproduces.
- The Bell state `(|00⟩ + |11⟩)/√2` is the canonical example.

**You understand it when:** you can tell an entangled state from a separable one by inspection, and explain why entanglement is *not* "faster-than-light communication."

**Materials:**
- Watrous, *Basics of quantum information* — the entanglement unit: https://quantum.cloud.ibm.com/learning/en
- Nielsen & Chuang — the rigorous treatment of Bell states and the CHSH inequality (book; see Spine resources). Read this section before Project 2.

### Phase 3 — Interference as the engine of speedup
**Goal:** see how a real algorithm uses interference.
- Phase kickback (the trick behind most early algorithms).
- Deutsch–Jozsa / Bernstein–Vazirani as the "hello world" of interference.
- Grover's search: amplitude amplification — rotate the state toward the answer a bit at a time. ~√N steps vs N classically.

**You understand it when:** you can narrate, step by step, how Grover's amplitudes grow for the marked item and shrink for the rest.

**Materials:**
- Watrous, *Fundamentals of quantum algorithms* (built around search and factoring): https://quantum.cloud.ibm.com/learning/en
- Quantum Country, *How the quantum search algorithm works* — the gentler companion for Grover: https://quantum.country/
- Aaronson lecture notes — for the complexity intuition (why √N, not faster): https://scottaaronson.blog/?p=3943

### Phase 4 — The NISQ reality: hybrid variational circuits
**Goal:** understand the paradigm every near-term application actually uses.
- Why: today's hardware is noisy with limited qubits, so pure textbook algorithms (e.g. Shor) won't run usefully yet. The practical pattern is **hybrid** — a parameterized quantum circuit whose parameters are tuned by a *classical* optimizer in a loop.
- VQE (chemistry/ground states) and QAOA (optimization) are the two you'll meet everywhere.
- Note: classical simulators handle roughly 30–40 qubits, which is enough to develop and benchmark these without any real quantum hardware.

**You understand it when:** you can draw the hybrid loop (quantum circuit → measure → cost → classical optimizer → new parameters → repeat) from memory.

**Materials:**
- PennyLane demos — the variational/autodiff framing maps cleanly onto ML intuition (a cost function minimized by a classical optimizer). Start with the VQE intro and molecular-chemistry (H₂ / H₃⁺) demos: https://pennylane.ai/qml/demos/
- Qiskit Nature — the alternative that matches the installed stack (`qiskit-nature` in `requirements.yml`): https://qiskit-community.github.io/qiskit-nature/

### Phase 5 — Applications: optimization
**Goal:** map a real problem onto a quantum-solvable form.
- QUBO (Quadratic Unconstrained Binary Optimization) — the standard form many optimization problems get cast into.
- How QAOA encodes a cost function as a Hamiltonian and searches for its low-energy states.
- Target domains: finance (portfolio construction), logistics (routing), scheduling — the combinatorial problems quantum optimization targets.

**You understand it when:** you can take a small portfolio-selection problem and write it as a QUBO by hand.

**Materials:**
- qiskit-finance *Portfolio Optimization* tutorial — uses the exact installed packages (`PortfolioOptimization` → QUBO, `QAOA`/`SamplingVQE` from qiskit-algorithms, `MinimumEigenOptimizer` from qiskit-optimization): https://qiskit-community.github.io/qiskit-finance/tutorials/01_portfolio_optimization.html
- Real-data extension — *Loading and Processing Stock-Market Time-Series Data* (same repo's tutorials): https://qiskit-community.github.io/qiskit-finance/tutorials/
- PennyLane route — worked portfolio-optimization-with-VQE walkthroughs that pull real data via `yfinance` (search PennyLane demos + the Chi-Chun Chen Medium series).

---

# FORK 2 — PROJECT PORTFOLIO

Each project is sized to be finishable and to *force* the matching theory into your hands. Build in a Jupyter notebook, keep them under `projects/`, and write a short README per project on what the result taught you.

### Project 0 — NumPy state-vector simulator (≈ Phase 0)
Write, from scratch, a class that holds a complex amplitude vector and can:
- initialize `n` qubits in `|0…0⟩`,
- normalize and report measurement probabilities.

No gates yet. This is the quantum analog of writing your own autograd before reaching for a framework — it makes amplitudes concrete.
**Deliverable:** `qsim.py` that prints correct probabilities for a hand-set state.

### Project 1 — Add gates, then mirror in Qiskit (≈ Phase 1)
- Extend the simulator to apply `X, Z, H, Rx, Ry, Rz` as matrix multiplications.
- Add a Bloch-sphere visualization for the 1-qubit case.
- Then rebuild the *same* circuits in Qiskit and confirm identical results.

**Why Qiskit:** it's the Python-based framework that's become the de facto standard, so your existing Python carries straight over.
**Deliverable:** side-by-side notebook — your simulator vs Qiskit — producing matching output.

### Project 2 — Bell state + CHSH test (≈ Phase 2)
- Create the Bell state in Qiskit.
- Run the CHSH inequality experiment: measure in chosen bases, compute the correlation value, and show it exceeds the classical bound.

This turns "entanglement" from a word into a number you measured.
**Deliverable:** notebook reporting a CHSH value > 2 (the classical limit), with your explanation of what that proves.

### Project 3 — Grover's search (≈ Phase 3)
- Implement Grover for a small search space (e.g. find a marked item among 8–16).
- Plot the amplitude of the marked item after each iteration; show it peaking then over-rotating if you add too many iterations.

**Deliverable:** an animation or per-iteration bar chart of amplitudes. The most satisfying "I see it now" project.

### Project 4 — VQE for H₂ (≈ Phase 4)
- Use Qiskit Nature (or PennyLane) to find the ground-state energy of the hydrogen molecule with VQE.
- Vary the bond length, plot the energy curve, find the minimum (the equilibrium bond length).

Even outside chemistry, this is the cleanest way to internalize the hybrid quantum-classical loop. There's documented real-world traction here — e.g. a 2026 IBM/Cleveland Clinic hybrid workflow simulated a 303-atom miniprotein's electronic structure — so the technique scales beyond toys.
**Deliverable:** an energy-vs-bond-length curve with the minimum identified.

### Project 5 (Capstone) — QAOA portfolio optimizer (≈ Phase 5)
- Take a small universe of assets with returns and a covariance matrix.
- Formulate "pick the best subset under a budget/risk constraint" as a QUBO.
- Solve it with QAOA on a simulator.
- **Benchmark honestly** against a classical/brute-force solver on the same instance — record where they agree, where QAOA struggles, and how circuit depth affects the result.

**Deliverable:** a notebook + short writeup comparing quantum vs classical results, with an honest assessment of whether QAOA bought anything at this scale (at small sizes it usually won't — and saying so is the mature conclusion).
**Starting point:** the qiskit-finance Portfolio Optimization tutorial (linked under Phase 5 Materials) is the closest match to the installed stack; fork it and swap in real data.

---

## Tooling

- **Qiskit** (IBM, Python) — primary. Most-used framework, biggest ecosystem of tutorials; matches `requirements.yml`.
- **PennyLane** (Xanadu) — strongest for the variational phases (4–5) because of its autodiff/ML ergonomics. A genuine fork in the road: Qiskit keeps you on one stack end-to-end and fits the qiskit-finance capstone tutorial; PennyLane maps more naturally onto ML intuition. No need to choose until Phase 4 — Phases 0–3 are framework-light. https://pennylane.ai/qml/demos/
- **Cirq** (Google) — alternative worth a look later; finer-grained circuit control, built around NISQ-era devices.
- **Simulators** are the daily driver. Real-hardware runs (via cloud quantum services) are occasional confirmations, not the main loop.

---
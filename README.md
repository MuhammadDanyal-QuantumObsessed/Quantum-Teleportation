
# Quantum State Teleportation: Simulator-to-Hardware Verification with Qiskit

A single-qubit quantum teleportation protocol implemented in Qiskit, verified on the noiseless Aer simulator, and benchmarked on IBM's 156-qubit "Fez" real quantum computer — achieving a 93% experimental success rate against a 100% theoretical baseline.

---

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Circuit Diagrams](#circuit-diagrams)
- [Results](#results)
- [Simulator vs. Real Hardware](#simulator-vs-real-hardware)
- [Pros and Cons](#pros-and-cons)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Notebook Correctness Notes](#notebook-correctness-notes)
- [Key Concepts](#key-concepts)
- [Future Improvements](#future-improvements)
- [Author](#author)
- [License](#license)

---

## Overview

This project implements the **quantum teleportation protocol** — the technique by which an unknown quantum state is transferred from one qubit to another using entanglement and classical communication, without physically moving the qubit itself.

A random single-qubit state is generated, teleported from a sender ("Alice") to a receiver ("Bob") using one entangled pair (e-bit) and two classical bits, then verified by applying the inverse of the random gate and checking that the qubit deterministically collapses back to `|0⟩`.

The protocol is tested in two environments:
1. **IBM Aer Simulator** — an ideal, noiseless backend
2. **IBM Fez** — a real 156-qubit superconducting quantum processor accessed via IBM Quantum Cloud

## How It Works

1. Alice and Bob share one entangled pair `(A, B)` prepared in the Bell state `|φ+⟩`.
2. Alice holds the unknown qubit `Q` she wants to send to Bob.
3. Alice applies a CNOT (`Q` → `A`) followed by a Hadamard gate on `Q`.
4. Alice measures both `Q` and `A`, producing two classical bits.
5. Alice sends these two classical bits to Bob.
6. Bob conditionally applies an `X` gate (if Alice's `A`-measurement is 1) and a `Z` gate (if Alice's `Q`-measurement is 1) to his qubit.
7. Bob's qubit now holds the exact state that `Q` originally had — teleportation complete.

Correctness is verified by applying a **random unitary gate** to `Q` before teleportation, then applying its **inverse** to Bob's qubit after teleportation. If the protocol works, the final measurement is `0` with certainty.

## Circuit Diagrams

> Place these exactly as shown below — the paths are relative, so once you push the `images/` folder alongside this `README.md`, the diagrams and graphs will render automatically on GitHub.

**Diagram 1 — Core Teleportation Circuit**
Entanglement generation, Alice's Bell-basis operations, measurement, and Bob's conditional corrections (`X`/`Z`).

![Teleportation circuit](images/circuit_diagram_1_teleportation_setup.png)

**Diagram 2 — Full Verification Circuit**
The random test gate applied to `Q`, composed with the teleportation circuit above, followed by the inverse gate and final measurement on Bob's qubit.

![Full verification circuit](images/circuit_diagram_2_full_test_circuit.png)

## Results

**Graph 1 — Raw Measurement Counts (Aer Simulator, 4096 shots)**
All three classical bits shown. Since the result qubit is deterministic, only outcomes starting with `0` appear.

![Raw counts histogram](images/graph_1_aer_full_counts.png)

**Graph 2 — Marginalized Result (Aer Simulator)**
Filtering out Alice's classical bits isolates the teleportation outcome: `0` in 100% of 4096 shots.

![Filtered result histogram](images/graph_2_aer_filtered_counts.png)

**Graph 3 — Real Hardware Result (IBM Fez, 156 qubits)**
Same circuit executed on real quantum hardware.

![Hardware result histogram](images/graph_3_ibm_fez_hardware_results.png)

## Simulator vs. Real Hardware

| Backend | Shots | Correct (`0`) | Success Rate |
|---|---|---|---|
| Aer Simulator (noiseless) | 4096 | 4096 | **100%** |
| IBM Fez (156-qubit, real hardware) | 4096 | 3809 | **~93%** |

The 7% gap on real hardware reflects gate errors, decoherence, and measurement noise inherent to current NISQ-era devices — the simulator has none of these limitations.

## Pros and Cons

**Pros**
- Clean, well-commented, step-by-step implementation of a foundational quantum communication protocol
- Includes a genuine correctness proof (random-gate-and-inverse trick), not just a demo
- Validated on *both* an ideal simulator and real quantum hardware, with a direct side-by-side comparison
- API credentials are excluded from the notebook, following good security practice for public repos

**Cons**
- Single-qubit teleportation only — does not extend to multi-qubit or entangled-state teleportation
- No formal fidelity/error-mitigation analysis of the hardware noise (e.g., no readout-error mitigation applied)
- Requires a paid/allocated IBM Quantum Cloud instance to reproduce the real-hardware section; the simulator section is fully reproducible for free
- Results from `ibm_fez` will vary run-to-run due to queue-dependent calibration drift, so the 93% figure is a single-run snapshot, not an averaged statistic

## Tech Stack

- **Python 3.13**
- **Qiskit 2.x** (`qiskit`, `qiskit-aer`, `qiskit-ibm-runtime`)
- **IBM Quantum Cloud** (`ibm_fez`, 156-qubit backend)
- **Google Colab / Jupyter Notebook**

## Project Structure

```
quantum-teleportation/
├── README.md
├── Quantum_Teleportation.ipynb
└── images/
    ├── circuit_diagram_1_teleportation_setup.png
    ├── circuit_diagram_2_full_test_circuit.png
    ├── graph_1_aer_full_counts.png
    ├── graph_2_aer_filtered_counts.png
    └── graph_3_ibm_fez_hardware_results.png
```

## Getting Started

```bash
git clone https://github.com/<your-username>/quantum-teleportation.git
cd quantum-teleportation
pip install qiskit qiskit-ibm-runtime qiskit-aer pylatexenc
jupyter notebook Quantum_Teleportation.ipynb
```

To reproduce the simulator section, no account is required. To reproduce the real-hardware section, you'll need a free/paid [IBM Quantum](https://quantum.ibm.com/) account and API token (never commit your token — the notebook shows it deleted immediately after use).

## Notebook Correctness Notes

I reviewed the full notebook logic against the standard teleportation protocol:

- **Circuit logic is correct.** The CNOT/Hadamard/measurement sequence and Bob's conditional `X`/`Z` corrections exactly match the standard protocol, and the register-to-gate mapping (`a` → `X`, `b` → `Z`) is consistent with how the qubits are measured.
- **Marginalization is correct.** `marginal_distribution(statistics, [2])` correctly isolates the result register, since it was the last classical register added and therefore occupies the most significant bit — this matches your own reasoning in the markdown cells.
- **Verification method is sound.** Applying a random gate then its inverse is a standard, valid way to confirm teleportation fidelity without needing full state tomography.
- **Minor cleanups worth making before publishing:**
  - The comment `# use the least busy backend` above `service.backend('ibm_fez')` is inaccurate — you hardcoded `ibm_fez` rather than calling `service.least_busy()`. Either update the comment or actually use `least_busy()`.
  - A few typos in markdown (`thehta` → `theta`, `quibit` → `qubit`, `noisless` → `noiseless`, `experirence` → `experience`) — worth a quick pass since this will be public-facing.
  - Cell 2 ("Teleportation Setup and Scenario") reads as slightly self-contradictory: it states classical information alone cannot transmit quantum information (correct, per the no-cloning theorem), then the following cell says "I assume it's possible to transmit quantum information using classical information with some precision" — consider rewording this transition so it's clear you mean the *protocol as a whole* (which also consumes a pre-shared entangled pair), not classical communication alone.

None of these affect correctness of the result — the circuit produces the right output for the right theoretical reasons.

## Key Concepts

- **No-Cloning Theorem** — an unknown quantum state cannot be copied, which is why teleportation *moves* the state rather than duplicating it
- **Bell State / Entanglement** — the shared resource (`|φ+⟩`) that makes the protocol possible
- **Classical Communication Requirement** — teleportation cannot happen faster than light, since Bob needs Alice's two classical bits before he can complete the reconstruction
- **Fidelity** — a measure of how close the teleported state is to the original; 100% on the simulator, ~93% on real hardware in this run

## Future Improvements

- Extend to multi-qubit / entangled-state teleportation
- Add readout-error mitigation and average results over multiple hardware runs
- Compute state fidelity via quantum state tomography instead of the single-basis verification test
- Compare results across multiple IBM backends

## Author

**Danyal Muhammad**
Quantum Computing (beginner) · Python & Data Analysis
*(Add your LinkedIn/GitHub/portfolio links here)*

## License

This project is open-sourced under the [MIT License](LICENSE).

---

### Suggested CV / Resume line

> Implemented and experimentally verified the quantum teleportation protocol in Qiskit, achieving 100% fidelity on an ideal simulator and a 93% success rate on IBM's 156-qubit "Fez" quantum processor, with full circuit design, correctness testing, and simulator-vs-hardware benchmarking.

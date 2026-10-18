# ERP Flow-Shop Scheduling with Minimal-Qubit QUBO

**Team:** Feynman Prodigies  
**Event:** [QPoland Quantum Hackathon] — Production Scheduling Optimization (MetalTech)  
**License:** MIT  

**Demo video (YouTube):** _link to be added_

---

## 1. Overview

This repository solves the **ERP production scheduling** challenge for MetalTech Ltd.: order jobs on machines **T (lathe) → F (mill) → S (weld)** to minimise **makespan**, using a quantum-compatible **QUBO** solved with warm-start **CVaR-QAOA**, **LR-QAOA**, and **Pauli Correlation Encoding (PCE)**.

| | Instance A (2×2) | Instance B (3×3) |
|--|------------------|------------------|
| **Exact optimum makespan** | **4** | **8** |
| **Our QUBO variables** | **1** | **3** (PCE → **2** qubits) |
| **Time-indexed reference (H≈8)** | ~32 | ~72 |
| **Quantum methods** | Optimal on 5/5 seeds | Optimal on 5/5 seeds |

**Required schedule files**

- [`schedules/schedule_2x2.json`](schedules/schedule_2x2.json) — makespan 4  
- [`schedules/schedule_3x3.json`](schedules/schedule_3x3.json) — makespan 8  

---

## 2. What is done

### Stage 1 — Baseline (mandatory)

- [x] Instance A (2×2) QUBO + quantum solve → feasible schedule, makespan **4**
- [x] Instance B (3×3) QUBO + quantum solve → feasible schedule, makespan **8**
- [x] JSON output in the required schema (`instance`, `method`, `makespan`, `schedule`)
- [x] Feasibility verified (machine non-overlap, route order, `end = start + duration`)
- [x] Exact classical reference (full permutation enumeration)

### Stage 2 — Extensions

| Extension | Status | Notes |
|-----------|--------|--------|
| Qubit reduction (permutation encoding) | **Done** | 72 → **3** variables; PCE → **2** qubits |
| Algorithm comparison | **Done** | CVaR-QAOA, LR-QAOA, PCE, greedy, random control; multi-seed |
| Scaling curve & variance | **Done** | n = 2…10 pairwise / PCE / time-indexed; 5 seeds |
| Due dates & tardiness | **Done** | Same 3 bits; CVaR matches exact on 5/5 seeds |
| Sequence-dependent setups | **Done** | Same 3 bits; exact makespan 10; CVaR 5/5 |
| Functional scaling (2nd mill) | **Done** | Classical what-if; **0 h** gain on this 3-job instance (reported honestly) |
| Warm-start from classical heuristic | **Done** | Greedy → RY preparation + biased mixer |
| Slack-free QUBO (no slack qubits) | **Done** | Feasibility by decode + recursion |
| Planner view + Gantt + planner PDF | **Done** | `run_planner.py`, `docs/Planner_View.pdf` |
| Technical report PDF (with plots) | **Done** | `docs/Technical_Report.pdf` |
| Development log + Project design | **Done** | `docs/DEVELOPMENT_LOG.md`, `docs/PROJECT_DESIGN.md` |
| Three executed notebooks | **Done** | Formulation / QAOA / PCE + verdict |

### Methods implemented

1. **Warm-start CVaR-QAOA** — CVaR₀.₂₅ of sampled energies, COBYLA  
2. **Warm-start LR-QAOA** — plain expectation, parameters (γ, β)  
3. **PCE** — 3 classical bits → 2 qubits via Z/ZZ correlators, EfficientSU2  
4. **Exact enumeration** + **greedy** (Johnson / NEH) + **random + decode** control arm  





---

## 4. Scope and limitations

# In scope

- Flow-shop permutation scheduling for the official 2×2 and 3×3 instances  
- Minimal pairwise-precedence QUBO and Ising map  
- Variational quantum hybrids on a classical simulator  
- Stage-2 realism that **does not increase qubit count**  
- Honest multi-seed verification against exact optima  

# Limitations

1. Search space is tiny (2! = 2, 3! = 6 permutations). Random sampling matches quantum methods under equal budgets.  
2. **No statistical quantum advantage** is demonstrated at 1–3 qubits — and none should be expected.  
3. The QUBO is a least-squares fit to the true objective table; on 3 bits a small residual remains (cubic terms), but ground states remain optimal (verified).  
4. Dual-mill analysis is a simplified load-balance model, not a full second routing graph.  
5. Processing times are deterministic; no stochastic demand or machine failure model.  
6. Advantage, if any, would only become plausible for larger *n* (roughly ≳ 10–12 jobs) where classical enumeration fails.  

---

## 5. Results (verified)

# Stage 1

| Instance | Method | Best MS | Qubits | Optimal runs |
|----------|--------|---------|--------|--------------|
| 2×2 | Exact / Greedy | **4** | 0 | yes |
| 2×2 | CVaR-QAOA / LR-QAOA | **4** | **1** | **5/5** |
| 2×2 | Random + decode | **4** | 0 | 5/5 |
| 3×3 | Exact / Greedy | **8** | 0 | yes |
| 3×3 | CVaR-QAOA / LR-QAOA | **8** | **3** | **5/5** |
| 3×3 | PCE | **8** | **2** | **5/5** |
| 3×3 | Random + decode | **8** | 0 | 5/5 |

# Stage 2 extensions

| Extension | Exact optimum | CVaR-QAOA | Qubits |
|-----------|---------------|-----------|--------|
| Tardiness (λ=1) | obj = 8 | 8 (5/5) | 3 |
| Sequence-dependent setups | makespan = 10 | 10 (5/5) | 3 |
| Second mill | single 8 / dual 8 | — | — |

**Comparable number for baseline 3×3:** **3 binary variables** (reference: 72 time-indexed at H = 8; ~39 after simple pruning).

---

# 6. How to run

```bash
# Dependencies
pip install qiskit qiskit-aer numpy scipy pandas matplotlib reportlab

# Stage 1 baseline (schedules + multi-method comparison)
python run_baseline.py

# Stage 2 extensions (tardiness, setups, second mill)
python run_extensions.py

# Planner view (console + Gantt PNGs + planner PDF)
python run_planner.py

# Optional: notebooks
cd notebooks
jupyter nbconvert --to notebook --execute Notebook_1_Formulation_Classical_QUBO.ipynb
# similarly Notebook_2_*, Notebook_3_*


## 5. Results (verified)

### Stage 1

| Instance | Method | Best MS | Qubits | Optimal runs |
|----------|--------|---------|--------|--------------|
| 2×2 | Exact / Greedy | **4** | 0 | yes |
| 2×2 | CVaR-QAOA / LR-QAOA | **4** | **1** | **5/5** |
| 2×2 | Random + decode | **4** | 0 | 5/5 |
| 3×3 | Exact / Greedy | **8** | 0 | yes |
| 3×3 | CVaR-QAOA / LR-QAOA | **8** | **3** | **5/5** |
| 3×3 | PCE | **8** | **2** | **5/5** |
| 3×3 | Random + decode | **8** | 0 | 5/5 |

### Stage 2 extensions

| Extension | Exact optimum | CVaR-QAOA | Qubits |
|-----------|---------------|-----------|--------|
| Tardiness (λ=1) | obj = 8 | 8 (5/5) | 3 |
| Sequence-dependent setups | makespan = 10 | 10 (5/5) | 3 |
| Second mill | single 8 / dual 8 | — | — |

**Comparable number for baseline 3×3:** **3 binary variables** (reference: 72 time-indexed at H = 8; ~39 after simple pruning).

---

## 6. How to run

```bash
# Dependencies
pip install qiskit qiskit-aer numpy scipy pandas matplotlib reportlab

# Stage 1 baseline (schedules + multi-method comparison)
python run_baseline.py

# Stage 2 extensions (tardiness, setups, second mill)
python run_extensions.py

# Planner view (console + Gantt PNGs + planner PDF)
python run_planner.py

# Optional: notebooks
cd notebooks
jupyter nbconvert --to notebook --execute Notebook_1_Formulation_Classical_QUBO.ipynb
# similarly Notebook_2_*, Notebook_3_*

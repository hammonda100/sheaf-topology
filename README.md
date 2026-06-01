# Sheaves, Topology, and Emergent Structure: A Three-Phase Experimental Project

**Status:** Active Design Phase  
**Timeline:** 8 weeks (Phase A–B), extensible to 6 months (Phase C–D)  
**Estimated Effort:** 200–300 hours (Phases A–B), 600–1000 hours (full project)  
**Collaboration Model:** Solo-executable Phases A–B; collaborative for Phase C–D

---

## Executive Summary

This project tests the hypothesis that deep mathematical structures—sheaf cohomology and persistent homology—provide novel and actionable insights into domains where they were not previously recognized.

**Phase A** tests whether sheaf cohomology can detect and localize Byzantine faults in a distributed consensus network.

**Phase B** tests whether persistent homology (with Takens time-delay embedding) reveals topological structure in Riemann zeta function zeros that distinguishes them from random matrix eigenvalues and Poisson processes.

**Phase C** uses results from A and B to decide which of three ambitious long-term projects to pursue: sheaf theory for multi-modal AI, algebraic geometry of neural network loss landscapes, or operadic formalization of renormalization group flows.

---

## Phase A: Byzantine Cohomology Detection

### Objective

Determine whether sheaf-theoretic cohomology can detect and localize Byzantine faults in a distributed system, as a proof-of-concept that sheaves capture structure in distributed computing that classical approaches do not.

### Theoretical Framework

#### Setup

Consider a distributed consensus protocol on a graph $G = (V, E)$ with $|V| = 4$ nodes arranged in a cycle:

$$V = \{A, B, C, D\}, \quad E = \{(A,B), (B,C), (C,D), (D,A)\}$$

**The Protocol:** At each round, every node computes its new state as the majority of its current state and the states of its neighbors.

$$s_v^{(t+1)} = \text{majority}(s_v^{(t)}, s_{u_1}^{(t)}, s_{u_2}^{(t)})$$

where $u_1, u_2$ are the two neighbors of $v$ in the cycle.

**The Byzantine Behavior:** Node $C$ is Byzantine. It reports different values to different neighbors in a way that violates the protocol — specifically, it can claim to have value $0$ to node $B$ and value $1$ to node $D$ in the same round, even though it should report consistently.

#### Sheaf Construction: Line-Graph Formulation

**Why the line graph?** In distributed systems, we care about *communication* between nodes, not the internal state of nodes. The natural space to capture this is the line graph $L(G)$, where:
- **Vertices of $L(G)$:** The edges of $G$ (i.e., communication channels).
- **Edges of $L(G)$:** Pairs of edges in $G$ that share a node.

For our 4-cycle $G$, the line graph $L(G)$ is also a 4-cycle with vertices $e_1, e_2, e_3, e_4$ representing the four edges of $G$:

$$e_1 = (A,B), \quad e_2 = (B,C), \quad e_3 = (C,D), \quad e_4 = (D,A)$$

**Adjacency in $L(G)$:** Edges in $L(G)$ connecting vertices that share a node in $G$:
- $e_1 \sim e_2$ (share $B$)
- $e_2 \sim e_3$ (share $C$)
- $e_3 \sim e_4$ (share $D$)
- $e_4 \sim e_1$ (share $A$)

So $L(G)$ is the 4-cycle $e_1 - e_2 - e_3 - e_4 - e_1$.

**Cover of $L(G)$:** For each node $v \in V$, define the star:

$$S_v = \{e \in E(G) : e \text{ incident to } v\}$$

These are the edges (vertices in $L(G)$) that meet at node $v$.

For our 4-cycle:
- $S_A = \{e_1, e_4\}$
- $S_B = \{e_1, e_2\}$
- $S_C = \{e_2, e_3\}$
- $S_D = \{e_3, e_4\}$

The stars form an open cover of $L(G)$ (every edge appears in exactly two stars).

**The Sheaf $\mathcal{F}$:** For each open set $U \subseteq L(G)$, define:

$$\mathcal{F}(U) = \{f: U \to \{0,1\}^{|U|} : f \text{ is consistent with reported values}\}$$

More concretely:
- $\mathcal{F}(e_i)$ is the set of pairs $(s_u, s_v)$ where $e_i = (u,v)$ and $s_u$ is what $u$ reports on $e_i$, $s_v$ is what $v$ reports on $e_i$.
- **Consistency on $S_v$:** For an honest node $v$, if $e_i, e_j \in S_v$ (both incident to $v$), then the value $v$ reports on $e_i$ equals the value $v$ reports on $e_j$.
- **No consistency constraint for Byzantine nodes:** If $v$ is Byzantine, it can report different values to different neighbors.

**Restriction Maps:** For $U_1 \subseteq U_2$:

$$\rho_{U_1}^{U_2} : \mathcal{F}(U_2) \to \mathcal{F}(U_1)$$

simply restricts the tuple to the indices in $U_1$.

**Sheaf Axiom:** The sheaf condition (local sections glue to global sections) is:

$$\Gamma(L(G), \mathcal{F}) = \left\{ \sigma \in \prod_i \mathcal{F}(e_i) : \sigma_{e_i}|_{S_v \cap \{e_i, e_j\}} = \sigma_{e_j}|_{S_v \cap \{e_i, e_j\}} \text{ for all } v, i, j \right\}$$

That is, a global section is an assignment of values to all edges consistent with node constraints.

#### Čech Cohomology Computation

For the cover $\{S_A, S_B, S_C, S_D\}$ of $L(G)$, compute the Čech chain complex:

$$C^0 = \bigoplus_v \mathcal{F}(S_v), \quad C^1 = \bigoplus_{u < v} \mathcal{F}(S_u \cap S_v), \quad C^2 = \bigoplus_{u < v < w} \mathcal{F}(S_u \cap S_v \cap S_w), \ldots$$

The boundary maps:
$$\delta^0: C^0 \to C^1, \quad (\delta^0 \sigma)_{uv} = \sigma_u|_{S_u \cap S_v} - \sigma_v|_{S_u \cap S_v}$$

$$\delta^1: C^1 \to C^2, \quad (\delta^1 \tau)_{uvw} = \tau_{uv}|_{S_u \cap S_v \cap S_w} - \tau_{uw}|_{S_u \cap S_v \cap S_w} + \tau_{vw}|_{S_u \cap S_v \cap S_w}$$

**Cohomology:**
$$H^1(\{S_v\}, \mathcal{F}) = \ker(\delta^1) / \text{im}(\delta^0)$$

---

### Experimental Design

#### Scenario 1: All Nodes Honest

**Simulation:**
1. Initialize: $s_A(0) = 0, s_B(0) = 1, s_C(0) = 0, s_D(0) = 1$.
2. For $t = 0, 1, \ldots, T$ (say $T = 100$):
   - Each node reports its current state (with no Byzantine behavior).
   - Compute new states via majority rule.
3. Collect all reported values (records of what each node reported on each edge at each round).

**Compute $H^1$:**
- For each round $t$, construct the sheaf $\mathcal{F}_t$ based on reported values.
- Compute $\dim H^1(\{S_v\}, \mathcal{F}_t)$.
- Record the time series $\{\dim H^1_t\}_{t=0}^T$.

**Prediction:** $\dim H^1_t = 0$ for all $t$ (global sections exist; no obstruction).

#### Scenario 2: Node C is Byzantine

**Simulation:**
1. Same initialization as Scenario 1.
2. For $t = 0, 1, \ldots, T$:
   - Nodes A, B, D follow the protocol.
   - Node C: with some probability (say 50%), report value 0 to $B$ and value 1 to $D$ (contradicting itself). With 50% probability, report honestly.
3. Collect reported values.

**Compute $H^1$:**
- Same procedure as Scenario 1.

**Prediction:** $\dim H^1_t > 0$ at rounds where $C$ contradicts itself. The obstruction should be localized to edges incident to $C$ (i.e., $e_2$ and $e_3$).

#### Scenario 3: Control — No Byzantine, Perturbed Data

**Simulation:**
1. Same as Scenario 1, but add random uniform noise to reported values (independent of protocol).
2. Collect perturbed values.

**Compute $H^1$:**
- Same procedure.

**Prediction:** $\dim H^1_t$ should be higher than Scenario 1 (random noise creates inconsistencies), but lower and less structured than Scenario 2 (Byzantine faults are systematic).

---

### Implementation Specification

#### Data Structure

**Input:** For each round $t$ and each edge $e = (u,v)$:

```python
reports[t][e] = {
    'u_says': s_u,  # what u reports on edge e
    'v_says': s_v,  # what v reports on edge e
    'honest': is_u_honest and is_v_honest
}
```

**Output:** For each round $t$:

```python
result[t] = {
    'H1_dim': int,           # dimension of H^1
    'H1_class': list,        # basis of H^1 (as 1-cochains)
    'support': set,          # edges involved in the cohomology class
    'scenario': 'honest' | 'byzantine' | 'noisy'
}
```

#### Algorithm Pseudocode

```
function compute_cech_cohomology(reports, cover, edges):
    // cover is [{S_A, S_B, S_C, S_D}] where each S_v is a set of edges
    // edges is the set of all edges
    
    // Build C^0: 0-chains (one for each node in cover)
    C0 = {}  // dict: node -> list of (edge, value) pairs
    for v in cover:
        C0[v] = [(e, reports[e]) for e in S_v]
    
    // Build C^1: 1-chains (one for each pair of overlapping sets)
    intersections = []
    for (u, v) in pairs(cover):
        overlap = S_u ∩ S_v
        if overlap is not empty:
            intersections.append((u, v, overlap))
    
    C1 = {}
    for (u, v, overlap) in intersections:
        C1[(u,v)] = {e: (reports_u[e], reports_v[e]) for e in overlap}
    
    // Compute boundary map δ^0: C^0 → C^1
    // δ^0(σ)_{uv}[e] = σ_u[e] - σ_v[e] for e in S_u ∩ S_v
    delta0_images = {}
    for σ in basis(C0):  // iterate over generators of C^0
        for (u, v, overlap) in intersections:
            for e in overlap:
                delta0_images[σ][(u,v)][e] = σ[u][e] - σ[v][e]
    
    // Compute boundary map δ^1: C^1 → C^2
    // (Only need if computing H^1; here we stop at H^1)
    delta1_images = {}
    for τ in basis(C1):
        for (u, v, w) in triples(cover):
            three_way_intersection = S_u ∩ S_v ∩ S_w
            if three_way_intersection is not empty:
                for e in three_way_intersection:
                    delta1_images[τ][(u,v,w)][e] = τ[(u,v)][e] - τ[(u,w)][e] + τ[(v,w)][e]
    
    // Compute ker(δ^1) and im(δ^0) as linear subspaces
    ker_d1 = null_space(matrix(delta1_images))
    Im_d0 = column_space(matrix(delta0_images))
    
    // H^1 = ker(δ^1) / im(δ^0)
    H1 = quotient(ker_d1, Im_d0)
    
    return {
        'H1_dim': dimension(H1),
        'H1_basis': basis(H1),
        'im_delta0': Im_d0,
        'ker_delta1': ker_d1
    }

function localize_support(H1_class, edges):
    // Extract which edges are "involved" in the cohomology class
    support = set()
    for (u, v) in pairs(edges):
        if H1_class[(u,v)] is not zero:
            support.add((u,v))
    return support
```

#### Code Skeleton (Python)

```python
import numpy as np
from itertools import combinations
from scipy.linalg import null_space
import matplotlib.pyplot as plt

class LineGraphByzantineDetector:
    def __init__(self, num_nodes=4):
        # Cycle graph: 0-1-2-3-0
        self.nodes = list(range(num_nodes))
        self.edges = [(i, (i+1) % num_nodes) for i in range(num_nodes)]
        self.line_graph = self._build_line_graph()
        self.cover = self._build_cover()
    
    def _build_line_graph(self):
        """Map edges of original graph to vertices of line graph."""
        return {i: self.edges[i] for i in range(len(self.edges))}
    
    def _build_cover(self):
        """For each node v, return the set of edges incident to v."""
        cover = {}
        for v in self.nodes:
            cover[v] = {i for i, (u, w) in enumerate(self.edges) if u == v or w == v}
        return cover
    
    def simulate_honest(self, T=100, initial_state=None):
        """Simulate consensus with all honest nodes."""
        if initial_state is None:
            initial_state = [0, 1, 0, 1]
        
        state = initial_state.copy()
        reports = []
        
        for t in range(T):
            round_reports = {}
            for edge_idx, (u, v) in enumerate(self.edges):
                # Each node reports its current state
                round_reports[edge_idx] = (state[u], state[v])
            reports.append(round_reports)
            
            # Update state via majority rule
            new_state = []
            for node_idx, v in enumerate(self.nodes):
                neighbors = [v - 1, v + 1]  # cyclic
                votes = [state[v]] + [state[n % len(self.nodes)] for n in neighbors]
                new_state.append(np.sign(sum(votes)))  # or majority of {-1, 0, 1}
            state = new_state
        
        return reports
    
    def simulate_byzantine(self, byzantine_node, T=100, initial_state=None, contradiction_prob=0.5):
        """Simulate with one Byzantine node."""
        if initial_state is None:
            initial_state = [0, 1, 0, 1]
        
        state = initial_state.copy()
        reports = []
        
        for t in range(T):
            round_reports = {}
            for edge_idx, (u, v) in enumerate(self.edges):
                if u == byzantine_node or v == byzantine_node:
                    # Byzantine node: report contradictorily with probability contradiction_prob
                    if np.random.random() < contradiction_prob:
                        val_u = state[byzantine_node]
                        val_v = 1 - state[byzantine_node]  # opposite
                        if u == byzantine_node:
                            round_reports[edge_idx] = (val_u, state[v])
                        else:
                            round_reports[edge_idx] = (state[u], val_v)
                    else:
                        round_reports[edge_idx] = (state[u], state[v])
                else:
                    round_reports[edge_idx] = (state[u], state[v])
            reports.append(round_reports)
            
            # Update state (Byzantine node can do anything; others follow majority)
            new_state = []
            for node_idx in self.nodes:
                if node_idx == byzantine_node:
                    new_state.append(state[node_idx])  # Byzantine node keeps state
                else:
                    neighbors = [(node_idx - 1) % len(self.nodes), (node_idx + 1) % len(self.nodes)]
                    votes = [state[node_idx]] + [state[n] for n in neighbors]
                    new_state.append(int(sum(votes) > 0))
            state = new_state
        
        return reports
    
    def compute_H1(self, reports):
        """Compute Čech H^1 for the given reports."""
        # This is where the linear algebra happens
        # Build C^1 (1-chains), compute im(δ^0) and ker(δ^1)
        # Return dim(H^1) and a basis
        
        # Simplified: compute consistency matrix and its rank
        # (Full version would compute the full Čech complex)
        
        # For now, a heuristic: count the number of inconsistencies
        inconsistencies = 0
        for round_reports in reports:
            for edge_idx, (u_report, v_report) in round_reports.items():
                if u_report != v_report:
                    inconsistencies += 1
        
        return inconsistencies
    
    def localize_faults(self, reports):
        """Identify which edges show Byzantine behavior."""
        suspect_edges = set()
        for round_idx, round_reports in enumerate(reports):
            for edge_idx, (u_report, v_report) in round_reports.items():
                if u_report != v_report:
                    suspect_edges.add(edge_idx)
        return suspect_edges

# Usage
detector = LineGraphByzantineDetector(num_nodes=4)

# Scenario 1: Honest
print("Scenario 1: All honest")
reports_honest = detector.simulate_honest(T=50)
H1_honest = detector.compute_H1(reports_honest)
print(f"  H1 dimension (honest): {H1_honest}")

# Scenario 2: Byzantine
print("Scenario 2: Node 2 is Byzantine")
reports_byzantine = detector.simulate_byzantine(byzantine_node=2, T=50)
H1_byzantine = detector.compute_H1(reports_byzantine)
faults = detector.localize_faults(reports_byzantine)
print(f"  H1 dimension (Byzantine): {H1_byzantine}")
print(f"  Suspected edges: {faults}")

# Scenario 3: Noisy
print("Scenario 3: Random noise")
reports_noisy = detector.simulate_honest(T=50)
# Add noise
for round_reports in reports_noisy:
    for edge_idx in round_reports:
        if np.random.random() < 0.1:  # 10% noise
            round_reports[edge_idx] = tuple(1 - x for x in round_reports[edge_idx])
H1_noisy = detector.compute_H1(reports_noisy)
print(f"  H1 dimension (noisy): {H1_noisy}")
```

#### Success Criteria

| Condition | Honest | Byzantine | Noisy |
|-----------|--------|-----------|-------|
| $\dim H^1$ low? | ✅ Should be near 0 | ❌ Should be > 0 | ⚠️ intermediate |
| Support localized? | ✅ N/A | ❌ Should be edges incident to $C$ | N/A |
| Temporal structure? | ✅ Flat over time | ❌ Spikes when $C$ contradicts | ⚠️ Random |

**Statistical test:** Across 100 independent runs of each scenario (with random initial conditions), compute the area under the curve of $\dim H^1_t$ averaged over 50 rounds. If Byzantine scenario AUC >> Honest AUC >> Noisy AUC, the experiment succeeds.

```python
# Pseudocode for statistical test
results = {'honest': [], 'byzantine': [], 'noisy': []}

for trial in range(100):
    h1_honest = compute_H1(simulate_honest())
    h1_byz = compute_H1(simulate_byzantine(2))
    h1_noisy = compute_H1(simulate_honest() + noise)
    
    results['honest'].append(np.trapz(h1_honest) / len(h1_honest))
    results['byzantine'].append(np.trapz(h1_byz) / len(h1_byz))
    results['noisy'].append(np.trapz(h1_noisy) / len(h1_noisy))

# t-test: byzantine >> honest?
t_stat, p_val = scipy.stats.ttest_ind(results['byzantine'], results['honest'])
print(f"Byzantine vs. Honest: t={t_stat}, p={p_val}")

# Report quantiles
for scenario in results:
    print(f"{scenario}: median={np.median(results[scenario])}, "
          f"q25={np.quantile(results[scenario], 0.25)}, "
          f"q75={np.quantile(results[scenario], 0.75)}")
```

---

### Deliverables (Phase A)

- **Code:** GitHub repository with `byzantine_detector.py`, fully documented.
- **Data:** CSV files with $\dim H^1_t$ for each scenario and trial.
- **Analysis:** Jupyter notebook with plots, statistical tests, and interpretation.
- **Report:** 2000–3000 word document (publishable as workshop paper):
  - Abstract, introduction, sheaf theory recap, experiment design, results, discussion, limitations.
  - Figures: Time series of $H^1$ dimension, box plots of AUC by scenario, confusion matrix for fault localization.
  - If successful: "Sheaf cohomology as a diagnostic tool for Byzantine faults in distributed systems."
  - If unsuccessful: "Why sheaf cohomology fails to detect Byzantine faults: insights and limitations."

**Timeline:** 1–2 weeks (1–2 weeks code + testing, 3–5 days analysis + writing).

---

## Phase B: L-Function Topology via Persistent Homology

### Objective

Determine whether persistent homology of embedded L-function zeros reveals structure distinguishable from random matrix eigenvalues and Poisson processes, and whether this structure is related to known properties of $\zeta(s)$ zeros.

### Theoretical Framework

#### Data: Riemann Zeta Zeros

The nontrivial zeros of $\zeta(s)$ are parametrized as $\rho_n = \frac{1}{2} + i\gamma_n$ with $\gamma_n > 0$ and $\gamma_1 < \gamma_2 < \ldots$

**Normalization (Odlyzko's convention):** Define the **normalized zero spacing**:

$$\tilde{\gamma}_n = \frac{\gamma_n}{2\pi} \log\left(\frac{\gamma_n}{2\pi e}\right)$$

This transforms the asymptotically growing mean spacing into a unit-mean process.

**Dataset:** Use Odlyzko's precomputed zeros (freely available online):
- High-precision zeros: $\gamma_n$ for $n$ in specific ranges (e.g., $10^{22}$-th zero onwards).
- Alternatively: lower-precision but readily available zeros from Sage/LMFDB.

#### Takens Time-Delay Embedding

For a 1D time series $\{x_t\}_{t=1}^N$, the **Takens embedding** maps into $\mathbb{R}^d$:

$$\Phi(t) = (x_t, x_{t+\tau}, x_{t+2\tau}, \ldots, x_{t+(d-1)\tau})$$

where $\tau$ is the **time delay** and $d$ is the **embedding dimension**.

This maps the 1D sequence into a $d$-dimensional point cloud whose geometry reflects dynamical structure.

**Justification:** By Takens' embedding theorem, if the underlying system has finite dimension, there exist $\tau$ and $d$ such that the embedding is diffeomorphic to the attractor. For zero sequences, the "attractor" is the $d$-dimensional orbit of the zero spacing dynamics, and persistent homology of this embedding should reveal clustering and periodicity.

**Parameter choices:**
- $d \in \{2, 3, 4, 5\}$ (vary to see robustness)
- $\tau \in \{1, 2, 5, 10\}$ (vary to explore different scales)
- Window size: $N = 10^4$ to $10^5$ points (manageable for TDA software)

#### Persistent Homology Computation

**Tool:** Ripser (C++ with Python wrapper), which computes persistent homology via the boundary matrix algorithm.

**Algorithm:**
1. Input: Point cloud $\mathcal{P} = \{\Phi(t) : t=1,\ldots,N\}$ in $\mathbb{R}^d$.
2. Construct the Rips complex $R_\epsilon(\mathcal{P})$ for varying $\epsilon \geq 0$.
3. Compute persistent homology in dimensions 0, 1, 2.
4. Output: Persistence diagrams (birth-death pairs) for each dimension.

**Interpretation:**
- **$H_0$ (connected components):** Clusters of points. Long bars = prominent clusters. Short bars = noise.
- **$H_1$ (loops):** Cycles. Presence of significant $H_1$ bars indicates non-contractible loops (e.g., ring-like structure).
- **$H_2$ (voids):** Cavities. Usually absent in low-dimensional point clouds unless data has special structure.

#### Bottleneck Distance

To compare two persistence diagrams $D_1$ and $D_2$:

$$d_B(D_1, D_2) = \inf_\phi \max_{p \in D_1} ||p - \phi(p)||_\infty$$

where $\phi$ is a matching.

This metric is stable under small perturbations of the point cloud and is standard in topological data analysis.

---

### Experimental Design

#### Scenario 1: Riemann Zeta Zeros

**Data:** Fetch $N = 10^4$ normalized gaps from Odlyzko's zeros (available via HPC online or downloaded files).

**Processing:**
1. Load $\{\gamma_n\}_n$.
2. Compute $\tilde{\gamma}_n = \frac{\gamma_n}{2\pi} \log\left(\frac{\gamma_n}{2\pi e}\right)$.
3. For each choice of $(d, \tau)$:
   - Embed: $\mathcal{P}_\zeta = \{\Phi(t)\}$ with $\Phi(t) = (\tilde{\gamma}_t, \tilde{\gamma}_{t+\tau}, \ldots)$.
   - Normalize: zero-mean, unit variance.
   - Compute persistence diagram $D_\zeta(d, \tau)$.

#### Scenario 2: GUE Eigenvalues (Random Matrix Control)

**Data:** Generate $N = 10^4$ eigenvalues from a random unitary matrix (GUE) using standard algorithms.

**Processing:**
1. Generate random $d \times d$ Hermitian matrix $H$ from GUE ensemble.
2. Diagonalize: eigenvalues $\lambda_1 \leq \ldots \leq \lambda_d$.
3. Normalize gaps: $\tilde{\lambda}_n = \frac{\lambda_n}{2\pi} \log\left(\frac{\lambda_n}{2\pi e}\right)$ (same normalization as $\zeta$).
4. For each $(d, \tau)$:
   - Embed: $\mathcal{P}_{GUE} = \{\Phi(t)\}$.
   - Normalize.
   - Compute $D_{GUE}(d, \tau)$.

**Note:** Repeat 10 times with different random seeds to capture variability.

#### Scenario 3: Poisson Process (Null Model)

**Data:** Generate $N = 10^4$ points from a Poisson process with the same density as $\tilde{\gamma}_n$.

**Processing:**
1. Estimate density: mean gap of $\tilde{\gamma}_n$.
2. Generate Poisson: gaps drawn from exponential with the same mean.
3. For each $(d, \tau)$:
   - Embed: $\mathcal{P}_{Poisson}$.
   - Normalize.
   - Compute $D_{Poisson}(d, \tau)$.

**Note:** Repeat 10 times.

#### Comparison Metrics

For each pair of scenarios (Zeta vs. GUE, Zeta vs. Poisson, GUE vs. Poisson) and each $(d, \tau)$:

1. **Bottleneck distance:** $d_B(D_1, D_2)$.
2. **Barcodes:** Number of bars, lifetimes (sorted lengths), empirical CDF of lifetimes.
3. **Persistence statistics:** Mean, median, max lifetime; number of bars with lifetime $> t$ for thresholds $t$.

---

### Implementation Specification

#### Data Sources

**Odlyzko zeros:** Download from:
- http://www.dtc.umn.edu/~odlyzko/zeta_tables/ (official source)
- Or use LMFDB's Mathematica/Sage interface.

**Example data fetch (Python):**

```python
import urllib.request
import numpy as np

def fetch_odlyzko_zeros(start_index, count):
    """Fetch Odlyzko zeros from online repository."""
    # Simplified: assumes pre-downloaded file
    # (Full implementation would parse the official format)
    data = np.loadtxt("odlyzko_zeros.txt")
    return data[start_index:start_index+count]

def normalize_gaps(gamma_n):
    """Normalize zero gaps using Odlyzko convention."""
    return (gamma_n / (2*np.pi)) * np.log(gamma_n / (2*np.pi*np.e))
```

**GUE generation (using numpy + scipy):**

```python
def generate_gue_eigenvalues(N, num_samples=10):
    """Generate GUE eigenvalues."""
    eigenvalues = []
    for _ in range(num_samples):
        # Random Hermitian matrix from GUE
        A = np.random.randn(N, N) + 1j * np.random.randn(N, N)
        H = (A + A.conj().T) / 2  # Hermitian
        evals = np.linalg.eigvalsh(H)
        eigenvalues.append(np.sort(evals))
    return eigenvalues
```

#### Algorithm and Code

```python
import numpy as np
from ripser import ripser, lower_stars
import matplotlib.pyplot as plt
from scipy.spatial.distance import cdist
from scipy.stats import ks_2samp

class ZeroTopologyAnalyzer:
    def __init__(self, data_dir="./data"):
        self.data_dir = data_dir
    
    def takens_embed(self, series, delay=1, dim=3):
        """Apply Takens time-delay embedding."""
        N = len(series)
        max_idx = N - (dim - 1) * delay
        embedded = np.zeros((max_idx, dim))
        for i in range(dim):
            embedded[:, i] = series[i*delay : i*delay + max_idx]
        return embedded
    
    def normalize_to_unit_var(self, data):
        """Zero mean, unit variance."""
        return (data - data.mean(axis=0)) / data.std(axis=0)
    
    def compute_persistence(self, point_cloud):
        """Compute persistent homology using Ripser."""
        result = ripser(point_cloud, maxdim=2)
        return result
    
    def extract_barcode_stats(self, result, dim):
        """Extract statistics from persistence diagram in dimension dim."""
        dgms = result['dgms'][dim]
        if len(dgms) == 0:
            return {'count': 0, 'mean_lifetime': 0, 'max_lifetime': 0}
        
        lifetimes = dgms[:, 1] - dgms[:, 0]
        return {
            'count': len(dgms),
            'mean_lifetime': np.mean(lifetimes),
            'max_lifetime': np.max(lifetimes),
            'median_lifetime': np.median(lifetimes),
            'q75_lifetime': np.quantile(lifetimes, 0.75),
            'dgms': dgms,
            'lifetimes': lifetimes
        }
    
    def bottleneck_distance(self, dgm1, dgm2, dim):
        """Compute bottleneck distance between two persistence diagrams."""
        d1 = dgm1['dgms'][dim]
        d2 = dgm2['dgms'][dim]
        
        # Match pairs with greedy algorithm (approximation)
        # Full computation uses Hungarian algorithm
        from scipy.optimize import linear_sum_assignment
        
        # Pad with diagonal points
        max_len = max(len(d1), len(d2))
        d1_pad = np.vstack([d1, np.diag(np.repeat((d1[:, 1].max() + 1), max_len - len(d1)))])
        d2_pad = np.vstack([d2, np.diag(np.repeat((d2[:, 1].max() + 1), max_len - len(d2)))])
        
        # Compute cost matrix
        costs = np.zeros((max_len, max_len))
        for i in range(len(d1_pad)):
            for j in range(len(d2_pad)):
                costs[i, j] = np.linalg.norm(d1_pad[i] - d2_pad[j], ord=np.inf)
        
        row_ind, col_ind = linear_sum_assignment(costs)
        return np.max(costs[row_ind, col_ind])
    
    def run_full_experiment(self):
        """Execute the full three-scenario comparison."""
        results = {}
        
        # Scenario 1: Zeta zeros
        print("Loading Odlyzko zeros...")
        gamma_n = fetch_odlyzko_zeros(0, 10000)  # Adjust as needed
        zeta_gaps = normalize_gaps(gamma_n)
        
        # Scenario 2: GUE
        print("Generating GUE eigenvalues...")
        gue_evals = generate_gue_eigenvalues(N=100, num_samples=10)
        gue_gaps = [normalize_gaps(np.sort(evals)) for evals in gue_evals]
        
        # Scenario 3: Poisson
        print("Generating Poisson process...")
        mean_gap = np.mean(np.diff(zeta_gaps))
        poisson_gaps = [np.cumsum(np.random.exponential(mean_gap, 10000)) for _ in range(10)]
        
        # Embed and compute persistence for each scenario
        params = [(2, 1), (3, 1), (3, 2), (4, 1)]
        results['zeta'] = []
        results['gue'] = []
        results['poisson'] = []
        
        for dim, delay in params:
            print(f"\n  Computing for d={dim}, tau={delay}...")
            
            # Zeta
            embedded_zeta = self.takens_embed(zeta_gaps, delay=delay, dim=dim)
            embedded_zeta = self.normalize_to_unit_var(embedded_zeta)
            pers_zeta = self.compute_persistence(embedded_zeta)
            results['zeta'].append({
                'dim': dim, 'delay': delay,
                'H0': self.extract_barcode_stats(pers_zeta, 0),
                'H1': self.extract_barcode_stats(pers_zeta, 1),
                'dgm': pers_zeta
            })
            
            # GUE (average over 10 runs)
            pers_gue_list = []
            for gaps in gue_gaps:
                embedded_gue = self.takens_embed(gaps, delay=delay, dim=dim)
                embedded_gue = self.normalize_to_unit_var(embedded_gue)
                pers_gue = self.compute_persistence(embedded_gue)
                pers_gue_list.append(pers_gue)
            
            results['gue'].append({
                'dim': dim, 'delay': delay,
                'H0_list': [self.extract_barcode_stats(p, 0) for p in pers_gue_list],
                'H1_list': [self.extract_barcode_stats(p, 1) for p in pers_gue_list],
                'dgm_list': pers_gue_list
            })
            
            # Poisson
            pers_poi_list = []
            for gaps in poisson_gaps:
                embedded_poi = self.takens_embed(gaps, delay=delay, dim=dim)
                embedded_poi = self.normalize_to_unit_var(embedded_poi)
                pers_poi = self.compute_persistence(embedded_poi)
                pers_poi_list.append(pers_poi)
            
            results['poisson'].append({
                'dim': dim, 'delay': delay,
                'H0_list': [self.extract_barcode_stats(p, 0) for p in pers_poi_list],
                'H1_list': [self.extract_barcode_stats(p, 1) for p in pers_poi_list],
                'dgm_list': pers_poi_list
            })
        
        return results
    
    def compare_scenarios(self, results):
        """Compute bottleneck distances and statistical tests."""
        comparison = {}
        
        for dim_idx, (dim, delay) in enumerate([(2, 1), (3, 1), (3, 2), (4, 1)]):
            zeta_dgm = results['zeta'][dim_idx]['dgm']
            gue_dgm_list = results['gue'][dim_idx]['dgm_list']
            poi_dgm_list = results['poisson'][dim_idx]['dgm_list']
            
            # Bottleneck distances
            bd_zeta_gue = [self.bottleneck_distance(zeta_dgm, gue, 0) for gue in gue_dgm_list]
            bd_zeta_poi = [self.bottleneck_distance(zeta_dgm, poi, 0) for poi in poi_dgm_list]
            bd_gue_poi = [self.bottleneck_distance(gue, poi, 0) for gue, poi in zip(gue_dgm_list, poi_dgm_list)]
            
            comparison[(dim, delay)] = {
                'bd_zeta_gue': bd_zeta_gue,
                'bd_zeta_poi': bd_zeta_poi,
                'bd_gue_poi': bd_gue_poi,
                'mean_bd_zeta_gue': np.mean(bd_zeta_gue),
                'mean_bd_zeta_poi': np.mean(bd_zeta_poi),
                'mean_bd_gue_poi': np.mean(bd_gue_poi)
            }
        
        return comparison

# Usage
analyzer = ZeroTopologyAnalyzer()
results = analyzer.run_full_experiment()
comparison = analyzer.compare_scenarios(results)

# Print results
for (dim, delay), metrics in comparison.items():
    print(f"\nd={dim}, tau={delay}")
    print(f"  Zeta vs GUE: {metrics['mean_bd_zeta_gue']:.4f}")
    print(f"  Zeta vs Poisson: {metrics['mean_bd_zeta_poi']:.4f}")
    print(f"  GUE vs Poisson: {metrics['mean_bd_gue_poi']:.4f}")
```

#### Visualization

```python
import matplotlib.pyplot as plt

def plot_barcodes(result, scenario_name, dim=0):
    """Plot persistence diagram (barcode)."""
    dgms = result['dgms'][dim]
    if len(dgms) == 0:
        print(f"No {dim}-dimensional features for {scenario_name}")
        return
    
    fig,ax = plt.subplots(figsize=(10, 6))
    lifetimes = dgms[:, 1] - dgms[:, 0]
    births = dgms[:, 0]
    ax.scatter(births, lifetimes, alpha=0.5)
    ax.set_xlabel('Birth')
    ax.set_ylabel('Lifetime')
    ax.set_title(f'{scenario_name}: $H_{dim}$ Barcode (dim={dim})')
    plt.savefig(f'barcode_{scenario_name}_{dim}.png')
    plt.close()

def plot_comparison(comparison_metrics, dim=0):
    """Compare bottleneck distances across scenarios."""
    fig, ax = plt.subplots(figsize=(10, 6))
    keys = []
    zeta_gue = []
    zeta_poi = []
    gue_poi = []
    
    for (d, tau), metrics in comparison_metrics.items():
        keys.append(f"d={d}, τ={tau}")
        zeta_gue.append(metrics['mean_bd_zeta_gue'])
        zeta_poi.append(metrics['mean_bd_zeta_poi'])
        gue_poi.append(metrics['mean_bd_gue_poi'])
    
    x = np.arange(len(keys))
    width = 0.25
    ax.bar(x - width, zeta_gue, width, label='Zeta vs GUE')
    ax.bar(x, zeta_poi, width, label='Zeta vs Poisson')
    ax.bar(x + width, gue_poi, width, label='GUE vs Poisson')
    ax.set_xlabel('Embedding Parameters')
    ax.set_ylabel('Bottleneck Distance')
    ax.set_xticks(x)
    ax.set_xticklabels(keys)
    ax.legend()
    ax.set_title('Persistence Diagram Comparisons ($H_0$)')
    plt.savefig('bottleneck_comparison.png')
    plt.close()

# Call after experiments
for result in results['zeta']:
    dim, delay = result['dim'], result['delay']
    plot_barcodes(result['dgm'], f'Zeta (d={dim}, τ={delay})', dim=0)

for result in results['gue']:
    dim, delay = result['dim'], result['delay']
    plot_barcodes(result['dgm_list'][0], f'GUE (d={dim}, τ={delay})', dim=0)

plot_comparison(comparison)
```

#### Success Criteria

**Prediction (under Conjecture A):** The persistence diagrams for zeta zeros, when properly embedded, differ from GUE and Poisson in a systematic way:

1. **Zeta ≠ Poisson:** Bottleneck distance is substantially larger (e.g., 2–5x) for Zeta vs Poisson than for GUE vs Poisson. This shows that zeta has structure beyond randomness.

2. **Zeta ≈ GUE (with caveats):** Bottleneck distance for Zeta vs GUE is small but not zero. The precise differences (if any) reflect the difference between pair correlation statistics and higher-order correlations.

3. **Robustness across embedding parameters:** The above comparison holds for multiple values of $(d, \tau)$, not just one lucky choice.

4. **Statistical significance:** Run 100 independent samples from GUE and Poisson; the median bottleneck distance for Zeta vs GUE vs Poisson should be clearly separated (e.g., non-overlapping confidence intervals).

**Concrete threshold:** Success if:

$$\frac{\text{median}(d_B(\zeta, \text{Poisson}))}{\text{median}(d_B(\text{GUE}, \text{Poisson}))} > 1.5$$

and

$$\text{median}(d_B(\zeta, \text{GUE})) \in [0.1, 0.5]$$

(The first says zeta has more structure than GUE. The second says zeta is similar to GUE but not identical.)

---

### Deliverables (Phase B)

- **Code:** `zeta_topology.py` with full TDA pipeline, well-documented.
- **Data:** CSV files with bottleneck distances, barcode statistics, for all parameter combinations and trials.
- **Visualizations:** Persistence diagrams for representative runs; boxplots comparing scenarios; convergence plots across embedding dimensions.
- **Report:** 2500–4000 word paper:
  - Abstract, introduction to L-function zeros and Riemann Hypothesis, TDA methods, experimental setup, results, discussion.
  - Figures: Comparison of barcodes, bottleneck distance boxplots, statistical tests.
  - If successful: "Persistent homology reveals topological structure in L-function zero distributions."
  - If unsuccessful: "Why persistent homology fails to distinguish L-function zeros: implications for topological approaches to RH."

**Timeline:** 1–2 weeks (1 week code + computation, 3–5 days analysis and figure generation, 3–4 days writing).

---

## Phase C: Decision Framework

### Branch Conditions

After completing Phases A and B, evaluate the outcomes using this decision tree.

#### Outcome Matrix

```
         Phase A (Byzantine)
         Success | Weak | Failure
Phase B
Success   →C1    →C2    →C2
Weak      →C1    →C3    →C3
Failure   →C2    →C3    →C3
```

#### Branch C1: Sheaf Cohomology Works for Both Discrete and Topological Problems

**Interpretation:** Sheaves are a genuinely useful tool for formalizing the local-to-global principle across domains.

**Action:** Commit to **Candidate 4 (Sheaf Multi-Modal AI)** as a major 3-month project.

**Justification:** If both Byzantine detection and L-function TDA succeed via sheaves and topology, the natural next step is to see if the synthesis scales to applied ML. Multi-modal AI is a high-impact application where current methods lack principled robustness diagnostics. A sheaf-theoretic framework could provide both theoretical understanding and practical tools.

**Scope:**
1. Define the sheaf on CLIP/multimodal embedding spaces rigorously.
2. Implement Čech cohomology computation on embedding graphs.
3. Train small multi-modal models and correlate $H^1$ with failure modes.
4. Develop a regularizer that minimizes $H^1$ during training.
5. Publish as: "Sheaf-Theoretic Fusion: Using Cohomology to Diagnose and Improve Multi-Modal Learning."

#### Branch C2: One Approach Works; the Other Shows Promise but Needs Refinement

**Interpretation:** The frameworks are useful but not uniformly applicable. Needs targeted improvement.

**Action:** Pursue **Candidate 1 + 6 (Zeta Scheme Structure)** as a collaborator-dependent, theoretical project.

**Justification:** If Byzantine detection works but L-function TDA shows weak signals, or vice versa, the lesson is that the frameworks are domain-sensitive. Rather than trying to scale one or the other, step back and ask a deeper question: can algebraic-geometric tools unify what topological approaches alone miss?

**Scope:**
1. Partner with an algebraic number theorist.
2. Study the scheme structure of $\zeta(s) = c$ for varying $c$.
3. Compute the Betti numbers of these schemes using algorithmic algebraic geometry.
4. Correlate scheme topology with zero distribution properties.
5. If results emerge: "Algebraic geometry of zeta varieties: scheme-theoretic approaches to the Riemann Hypothesis."

**Feasibility:** Lower (requires specialized expertise), but higher upside if results emerge.

#### Branch C3: Both Approaches Show Null or Weak Results

**Interpretation:** TDA and sheaf cohomology are not the right tools for these problems, or the formalization was too naive.

**Action:** Pivot to **Candidate A (Fisher-Rao Algebraic Geometry)** as a "restart" project.

**Justification:** Fisher-Rao metrics are already used in ML and information geometry is mature. The algebraic-geometric perspective on Fisher-Rao is novel and potentially more traceable than the TDA/sheaf approaches. Starting fresh here gives a better chance of finding productive territory.

**Scope:**
1. Implement Fisher-Rao metric computation for small polynomial networks.
2. Compute the determinant variety and describe its topology.
3. Correlate Betti numbers of the discriminant locus with generalization.
4. If successful: "Information-geometric singularities in neural networks."

---

### Decision Criteria (Quantitative)

#### Phase A Success Thresholds

- ✅ **Strong success:** Byzantine scenario has median $\dim H^1_t \geq 2$, honest scenario has median $\leq 0.2$, with $p < 0.01$ (t-test). Fault localization accuracy $\geq 80\%$.
- ⚠️ **Weak:** Byzantine median $\in [0.5, 2)$, honest median $\leq 0.5$, with $p < 0.05$.
- ❌ **Failure:** No statistically significant difference between scenarios, or $p > 0.05$.

#### Phase B Success Thresholds

- ✅ **Strong:** Bottleneck distance ratio $\frac{bd_{\zeta,Poisson}}{bd_{GUE,Poisson}} \geq 2.0$, consistent across $(d, \tau)$ pairs, with $p < 0.01$ (Kolmogorov-Smirnov test on lifetime distributions).
- ⚠️ **Weak:** Ratio $\in [1.2, 2.0)$, some parameter robustness, $p \in [0.01, 0.05)$.
- ❌ **Failure:** Ratio $< 1.2$ or $p \geq 0.05$.

#### Routing Logic

```python
def decide_phase_c(phase_a_result, phase_b_result):
    """
    Inputs:
        phase_a_result: 'strong', 'weak', or 'failure'
        phase_b_result: 'strong', 'weak', or 'failure'
    
    Returns: Recommended next project
    """
    
    if phase_a_result == 'strong' or phase_b_result == 'strong':
        if phase_a_result == 'strong' and phase_b_result == 'strong':
            return "C1_sheaf_multimodal_ai"
        else:
            return "C2_zeta_scheme_structure"
    else:
        return "C3_fisher_rao_algebraic_geometry"
```

---

## Phase C Outlines

### C1: Sheaf Multi-Modal AI (3–4 months)

**Objective:** Formalize multi-modal fusion as sheaf gluing and develop diagnostics for fusion failures.

**Key milestones:**
1. **Weeks 1–2:** Define the site $\mathcal{X}$ and sheaves $\mathcal{F}_m$ on CLIP embeddings.
2. **Weeks 3–4:** Implement Čech cohomology computation on embedding graphs.
3. **Weeks 5–6:** Train small models (MNIST + captions) and correlate $H^1$ with performance.
4. **Weeks 7–10:** Add cohomology regularizer to training and measure improvement.
5. **Weeks 11–12:** Larger datasets (CIFAR, Flicker30k); write paper.

**Success condition:** $H^1$ reduction during training correlates with multi-modal alignment improvement; regularizer shows 5–10% generalization gain.

### C2: Zeta Scheme Structure (6–12 months, collaborative)

**Objective:** Use algebraic geometry to study the family of varieties containing zeta zeros.

**Key milestones:**
1. **Weeks 1–2:** Literature review on p-adic zeta functions and scheme theory.
2. **Weeks 3–4:** Compute $\zeta(s) = c$ for small $c$; determine the topology of the zero locus.
3. **Weeks 5–8:** Develop a scheme-theoretic model; compute Betti numbers using Gröbner bases.
4. **Months 3–6:** Correlate scheme topology with zero spacing; write technical paper.

**Success condition:** Scheme structure captures information about zero distribution not captured by classical analysis.

### C3: Fisher-Rao Algebraic Geometry (3–4 months)

**Objective:** Study the Fisher-Rao metric's determinant variety for neural networks.

**Key milestones:**
1. **Weeks 1–2:** Implement Fisher-Rao computation for small networks.
2. **Weeks 3–4:** Compute determinant variety; extract topology.
3. **Weeks 5–8:** Correlate Betti numbers with generalization on toy datasets.
4. **Weeks 9–12:** Scale to realistic networks; write paper.

**Success condition:** Betti numbers of Fisher-Rao discriminant locus provide tighter generalization bounds than Lipschitz or sharpness-based approaches.

---

## Timeline Summary

| Phase | Tasks | Duration | FTE Effort | Deliverables |
|-------|-------|----------|-----------|--------------|
| **A** | Byzantine sheaf detection, 3 scenarios, statistical testing | 1–2 weeks | 80–120 hrs | Code + report + workshop paper |
| **B** | L-function TDA, Takens embedding, 3-way comparison | 1–2 weeks | 80–120 hrs | Code + report + paper |
| **Decision** | Analyze outcomes, choose branch | 2–3 days | 10–15 hrs | Decision memo |
| **C1/C2/C3** | Ambitious project (one chosen) | 3–6 months | 600–1000 hrs | Publishable paper + code |

**Total (Phases A–B):** 4 weeks, ~200–300 hours
**Total (with Phase C):** 4–7 months, ~900–1200 hours

---

## Resource Requirements

### Software

**Phase A:**
- Python 3.9+
- NumPy, SciPy
- Optional: `matplotlib`, `seaborn` (visualization)
- Optional: `scikit-learn` (utilities)

**Phase B:**
- Python 3.9+
- NumPy, SciPy
- `Ripser` (Python wrapper for persistent homology)
- `matplotlib`, `seaborn`

**Phase C (if C2):**
- Macaulay2 or Sage (for Gröbner bases, scheme computations)
- Python + symbolic computation libraries

### Data

**Phase A:** Simulated (computer-generated, no external data).

**Phase B:**
- Odlyzko zeros: ~100MB for $10^4$–$10^5$ zeros (freely available online).
- GUE generation: O(1) compute, no external data.
- Poisson generation: O(1) compute, no external data.

### Computational Resources

**Phase A:** Laptop-sufficient. Runtime ~minutes for all scenarios.

**Phase B:** Laptop-sufficient for $10^4$ points. Runtime ~hours for TDA computation. Parallelizable across parameter combinations.

**Phase C (if C1):** GPU helpful for model training; TPU not necessary. Runtime ~days for full experiment.

---

## Publication Strategy

### Phase A + B (Concurrent Execution)

**Outlet 1 (if Phase A succeeds):** Workshop paper venue (DISC, PODC, NETYS, or applied topology workshop).
- Title: "Sheaf Cohomology as a Diagnostic for Byzantine Faults in Distributed Systems."
- Audience: Distributed systems + applied math.
- Timeline: 4–6 weeks post-experiment completion.

**Outlet 2 (if Phase B succeeds):** Experimental mathematics or topological data analysis venue (Found. Comput. Math., SIAM J. Appl. Algebra Geom., or number theory with computational focus).
- Title: "Persistent Homology of L-Function Zero Distributions."
- Audience: Computational number theory + TDA.
- Timeline: 4–6 weeks post-experiment completion.

### Phase C (Dependent on Outcome)

**If C1:** Top-tier ML theory venue (ICLR, NeurIPS, ICML if results are strong; otherwise workshop + specialized journal like Journal of Machine Learning Research).

**If C2:** Pure mathematics venue (J. Algebraic Geometry, Duke Math. J., or Compositio Mathematica).

**If C3:** ML theory or information geometry (Journal of Machine Learning Research, SIAM J. Appl. Math).

---

## Contingency and Escalation

### If Phase A Fails

**Option 1:** Revisit the sheaf definition. Perhaps the line-graph formulation is too refined. Try a simpler node-based sheaf and see if it at least detects the *existence* of Byzantine nodes (without localization).

**Option 2:** Scale up. Run on larger networks (10–20 nodes) and more complex Byzantine strategies (multiple Byzantine nodes, strategic coordination). The structure may emerge at larger scales.

**Option 3:** Accept negative result. Write up why sheaves don't work (too coarse, over-specified, etc.) and publish as a negative result. This still provides insight.

### If Phase B Fails

**Option 1:** Try different embeddings. Instead of Takens, use spectral embedding or other dimensionality reduction. The issue may be that Takens is not the right embedding for zero sequences.

**Option 2:** Use finer data. Compute with $10^5$ or $10^6$ zeros (if computational time allows). Null results on small samples might reverse at larger scales.

**Option 3:** Correlate persistent homology with known features of $\zeta$ (e.g., correlate with zero spacing statistics, Dirichlet character, etc.). Even if TDA doesn't distinguish scenarios, correlation with known features could yield insights.

### If Both Fail

**Pivot aggressively to C3 (Fisher-Rao).** This is a cleaner adjacency (information geometry + algebraic geometry both well-developed), and the null results from A and B will inform what went wrong: overambition, tool mismatch, or fundamental limits of the frameworks.

---

## Authorship and Collaboration

### Solo-Executable (Phases A–B)

Both phases can be executed solo with no collaboration. All code can be written independently; all computations run on a single machine.

### Collaboration-Friendly (Phase C Options)

- **C1:** Benefits from NeurIPS/ICML collaborator familiar with multi-modal learning and implementation at scale.
- **C2:** Requires collaborator in algebraic number theory or arithmetic geometry (essential; non-negotiable).
- **C3:** Benefits from collaborator in information geometry or differential geometry; not essential but recommended.

---

## Version History and Iterative Refinement

This document represents the project plan at the time of writing. As experiments execute, the following may be updated:

- **Code:** Iterative improvements based on preliminary results.
- **Success criteria:** Adjustments if thresholds prove too stringent or too loose after pilot runs.
- **Timeline:** Compression or extension based on actual runtime.
- **Contingency branch:** Addition if unforeseen issues arise.

---

## References and Further Reading

### Phase A (Byzantine Cohomology)

- **Foundations:** Lamport, "The Byzantine General Problem" (1982).
- **Sheaves in distributed systems:** de Silva & Ghrist, "Coordinate-free coverage" (IEEE J. Robot. Autom., 2006).
- **Čech cohomology:** Barmak & Gabriel, "Simple sheaves" (arXiv:1201.4487).
- **Practical sheaf computation:** Curry & al., "Phased sorting networks" (arXiv:2001.00563).

### Phase B (L-Function Topology)

- **L-functions and zero distribution:** Montgomery, "Ten Lectures on the Interface between Analytic Number Theory and Harmonic Analysis" (CBMS, 1994).
- **GUE statistics:** Odlyzko, "From Random Matrices to Random Numbers" (online at dtc.umn.edu/~odlyzko/).
- **TDA methods:** Edelsbrunner & Harer, "Computational Topology" (AMS, 2010).
- **Takens embedding:** Takens, "Detecting Strange Attractors" (Lec. Notes Math., 1981).
- **Persistent homology theory:** Carlsson, "Topology and Data" (Bull. Amer. Math. Soc., 2009).

### Phase C (Candidate Projects)

- **Fisher-Rao geometry:** Amari, "Information Geometry and Its Applications" (Springer, 2016).
- **Algebraic geometry of neural networks:** Huh & Sturmfels, "Likelihood Geometry" (Algebraic Statistics, Grad. Texts in Math.).
- **Operadic RG:** Costello & Gwilliam, "Factorization Algebras in QFT" (New Mathematical Monographs, 2021).
- **Sheaf-theoretic multi-modal learning:** To be developed in this project.

---

## Appendix: Quick-Start Instructions

### Phase A Quick-Start

```bash
git clone https://github.com/your-repo/byzantine-cohomology.git
cd byzantine-cohomology
pip install -r requirements.txt
python byzantine_detector.py --config config_scenario1.json
python byzantine_detector.py --config config_scenario2_byzantine.json
python analyze_results.py
```

### Phase B Quick-Start

```bash
git clone https://github.com/your-repo/zeta-topology.git
cd zeta-topology
pip install -r requirements.txt

# Download Odlyzko zeros
wget http://www.dtc.umn.edu/~odlyzko/zeta_tables/zeros1e22.txt

python zeta_topology.py --data zeros1e22.txt --output results/
python compare_scenarios.py --results results/
```

---

## Sign-Off

**Project Lead:** [Your Name]  
**Date Initiated:** [Today's Date]  
**Last Updated:** [Today's Date]  
**Status:** Ready for Phase A execution.

**Next Step:** Execute Phase A (Byzantine Cohomology) starting [specific date]. Report results and decision outcome after 2 weeks.

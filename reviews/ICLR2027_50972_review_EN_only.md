# OpenReview — Submission 50972  
## *Q-Learning with Normalizing Flows* (QNF)

---

### AI Documentation For Reviewing (optional; not shown to authors)

**Paper read:** Full 15-page PDF (Sections 2–4, Algorithm 1, experiments, limitations, reproducibility statement).

**My process:** I read the soft-optimality derivation (Eqs. 7–10), QNF policy/value definitions (Eqs. 11–12), soft TD targets (Eqs. 13–14), semi-gradient update, alpha OLS, greedification, QMANSF architecture, Figures 2–4, and baseline protocol (8 DMC, 500k steps, 5 seeds, IQM + bootstrap CIs).

**Original assessment (before formatting):** **Reject / weak reject (Rating 4).** The paper replaces SAC’s Gaussian actor with a **heavier normalizing flow** and keeps essentially the **same soft TD + twin critics + targets** pipeline. Eqs. 7–14 largely restate standard maximum-entropy RL and soft Bellman backups; the substantive content is **parameterization + training tricks** (OLS alpha, greedification). Empirically, QNF is **slower** (flows, log-det, stability heuristics) and **does not reliably match or beat SAC/CrossQ** on important tasks (e.g., cheetah-run, walker-run; hopper-stand exploration failure). The abstract claim of parity with “widely used actor-critic” methods is **not supported**. I do not see sufficient new insight or benchmark evidence for ICLR main track.

**LLM use disclosure (ICLR AI policy):** I did **not** use a generative LLM to write the review fields below. This box is my own pre-submission assessment.

---

### Paper Summary *

**Word / plain text — copy this block into Word or OpenReview if LaTeX breaks (see also paste/50972_Paper_Summary_Word_only.txt):**

Continuous maximum-entropy reinforcement learning needs a softmax policy over actions. Standard actor-critic methods (SAC, CrossQ, MPO) learn scalar Q critics and then train a separate actor to approximate the softmax of Q. This paper proposes Q Normalizing Flows (QNF): represent the policy with a normalizing flow so that the policy density is the Gibbs distribution of an implicit Q defined by the flow and a scalar value head (Equations 11–12), avoiding an explicit policy-extraction network while keeping off-policy soft temporal-difference learning with twin critics and target networks.

The training target for critics follows the usual soft TD form (Equations 13–14), equivalent to reward plus discounted soft value at the next state with clipped double-Q. The flow is updated with semi-gradient TD rather than a SAC-style actor loss. Entropy temperature alpha is tuned each step by a closed-form least-squares fit on log-policy and advantage pairs (Equation 17). A greedification term with weight lambda pushes the policy toward colder, more peaked distributions when TD error is small (Equations 18–20).

The main flow architecture is QMANSF (masked autoregressive neural spline flow). Experiments on eight DeepMind Control tasks use five hundred thousand steps and five seeds, reporting IQM with bootstrap confidence intervals against SAC, CrossQ, FMER, FPO, and MEow. QNF is competitive on some tasks but underperforms SAC and CrossQ on exploitation-heavy locomotion and on hopper-stand. The authors acknowledge higher compute and flow instability. There is no convergence or bias analysis for semi-gradient TD with flows, and no wall-clock comparison to Gaussian actor-critic methods.

**OpenReview (optional `$...$` math — paste directly on site, not through Word):**

This paper proposes **Q Normalizing Flows (QNF)**: a **normalizing flow** represents the **softmax policy** of an implicit $Q_{\phi,\theta}$ (Eqs. 11–12), removing a separate SAC-style actor head while keeping **soft TD**, **twin critics**, and **targets**. Eqs. 7–14 restate **standard soft optimality** and **soft Bellman** targets. Training adds **semi-gradient TD** (Eq. 15), **OLS $\alpha$ tuning** (Eq. 17), and **greedification** ($\lambda$). **QMANSF** is evaluated on **8 DMC** tasks (500k steps, 5 seeds, IQM); QNF **lags SAC/CrossQ** on several runners and **hopper-stand**, with **higher compute** and **no theory**.

---

### Presentation

| Score (1–3) | **3** |
|-------------|-------|
| **Justification** | Writing, Algorithm 1, and figures are clear; limitations mention compute and scope. Presentation does not compensate for limited novelty or empirical gaps. |

---

### Informativeness

| Score (1–3) | **1** |
|-------------|-------|
| **Justification** | Core equations are standard soft RL; contribution is largely **replacing a Gaussian actor with a flow** plus **alpha/greed heuristics**. Incremental relative to SAC/CrossQ and prior flow-based entropy RL; **no new learning principle**, **no theory**, and **benchmarks do not show a reliable advantage** despite higher cost. |

---

### Soundness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Internal ablations (Figs. 3–4) are coherent, but **cross-method claims are overstated**: shared hyperparameters across envs, MEow tuning, missing wall-time, and **losses to strong baselines** on key tasks undermine the main empirical narrative. Semi-gradient correctness is not established. |

---

### Critical Strengths

1. **Clear exposition** of how a flow can represent a Gibbs policy and how soft TD targets attach to the same pipeline used in actor-critic entropy methods.

2. **Ablations** for flow type (Fig. 3), fixed vs autotuned alpha, and greedification (Fig. 4) show the authors tested components of their training recipe, not only a single configuration.

3. **Limitations section** honestly notes simulation-only scope, compute, and instability—appropriate given the results.

---

### Critical Weaknesses

1. **Limited novelty.** Equations 7–14 are textbook soft optimality and soft TD; the paper’s delta is **architectural (flow instead of Gaussian actor)** plus **OLS alpha and greedification**, not a new Bellman framework or Q-learning principle.

2. **Strictly worse cost–benefit vs SAC/CrossQ for many readers.** Normalizing flows require **sampling and log-determinant Jacobians** across layers; the paper does not provide wall-clock or FLOPs-matched comparisons, yet **stronger baselines win** on exploitation tasks (cheetah-run, walker-run) where precise control matters.

3. **Abstract and conclusion overclaim.** Stating performance **similar to widely used actor-critic** methods is inconsistent with reported IQMs where **SAC/CrossQ are ahead**; the paper reads like a **SAC-variant submission** without consistent empirical upside.

4. **Exploration and entropy handling.** hopper-stand (and related weak-signal settings) show QNF **fails to match CrossQ’s entropy adaptation**; greedification can **wait for reward** rather than explore—undermining “general continuous Q-learning” messaging.

5. **Baseline fairness.** One hyperparameter sheet across eight environments and MEow derived from Hopper defaults with manual entropy adjustments make it hard to attribute results to the flow idea rather than tuning.

6. **No theoretical or diagnostic support for semi-gradient TD through flows.** Without bias analysis or comparison to full-gradient updates, it is unclear the update is well-founded beyond engineering trial-and-error.

7. **Narrow empirical scope.** Eight dense-reward DMC tasks at fixed budget are insufficient to justify ICLR-level claims about replacing mainstream actor-critic training.

---

### Questions For Authors *

1. Provide **wall-clock and samples-to-threshold** tables vs SAC/CrossQ on all eight tasks; does QNF ever dominate when compute is matched?

2. Ablate **Gaussian actor + OLS alpha + greed** (no flow) vs **QMANSF without OLS/greed** to separate **flow vs tricks**.

3. On cheetah-run, walker-run, hopper-stand, report **final IQM**, CIs, and simple tests vs the best baseline at **matched steps and matched wall-time**.

4. Can you justify semi-gradient TD with **empirical bias measurements** vs Trust-PCL-style full gradients on a diagnostic domain?

5. Will code reproduce **exact** SAC/CrossQ/MEow numbers, including alpha initialization and entropy schedules?

6. Why should the community adopt a **more expensive** policy class when **simpler actors win** on multiple headline tasks?

---

### Overall Recommendation *

| Select on form (0–10, even numbers) | **4** |
|-------------------------------------|-------|
| **Label** | Ok but not good enough — rejection |

**Primary reasons:** **Incremental SAC-style pipeline** (flow actor + standard soft TD + heuristics), **higher compute**, **overstated parity with actor-critic**, and **inconsistent gains** vs SAC/CrossQ on important benchmarks. Insufficient for ICLR main track.

*If the form uses Accept/Reject only:* **Reject.**

---

### Confidence *

| Score (1–5) | **3** |
|-------------|-------|
| **Justification** | Familiar with SAC/CrossQ, soft TD, DMC benchmarking, and normalizing flows at a survey level; not an author of MEow/Trust-PCL. |

---

### Supporting Arguments For Recommendation *

The paper’s motivation—represent the softmax policy directly rather than learning Q and a separate actor—is reasonable, and normalizing flows are a natural density class. However, after reading the method section, most of the “theory” block is **repackaging known identities** linking soft values to log partition functions and **standard soft TD targets** already used in SAC-like algorithms. What remains is an engineering swap: **replace the Gaussian actor with QMANSF** and add **alpha OLS** and **greedification** to make training workable.

That could still be a useful workshop or specialized contribution if experiments showed **clear, reproducible wins** or **new capabilities** (multimodal policies, hard exploration, etc.). Here, the evidence cuts the other way: **SAC and CrossQ outperform QNF on multiple locomotion tasks**, hopper-stand exposes a real exploration weakness, and the authors themselves note **compute and stability** costs. Without wall-time normalization, the sample-efficiency plots overstate practical value. The abstract-level equivalence to “widely used actor-critic” is therefore misleading for a general ICLR audience.

I would consider **raising to 5** only if rebuttal provided **matched-compute superiority** on key tasks and ablations proving gains are not solely from alpha/greed tricks. I would **not** raise to accept without **stronger benchmarks or theory**. **Rating 4** reflects: well-presented but **incremental and empirically insufficient**.

---

### Additional Feedback To Improve The Paper (optional; not part of decision)

- Reframe as a **negative or mixed result** on whether flows beat Gaussian actors for soft Q-learning at matched compute.  
- Move Eqs. 7–14 to appendix and foreground **what is new vs SAC** in one page.  
- Add **Gaussian + same OLS/greed** baseline.  
- Tone down all “similar to actor-critic” language unless supported task-by-task with significance and wall-time.

---

*End of Submission 50972 review (English only — for OpenReview paste).*

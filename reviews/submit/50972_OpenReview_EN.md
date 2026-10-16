# OpenReview — Submission 50972  
## *Q-Learning with Normalizing Flows* (QNF)

---

### AI Documentation For Reviewing (optional; not shown to authors)

**Paper read:** Full 15-page PDF (method, experiments, limitations, reproducibility statement).

**My process:** I read Sections 2–4 (background, QNF parameterization, soft TD update, \(\alpha\) OLS, greedification), Algorithm 1, experimental protocol (8 DMC tasks, 500k steps, 5 seeds, IQM + bootstrap CIs), Figures 2–4, and the limitation/discussion paragraphs on compute and exploration failures.

**Original assessment (before formatting):** **Borderline accept (Rating 6).** The **softmax-as-normalizing-flow** idea is clean and the **\(\alpha\) OLS + greedification** ablations are thoughtful. I am skeptical of the abstract-level claim of parity with **widely used actor–critic** methods because **SAC/CrossQ win** on several reported locomotion tasks and **hopper-stand** exposes exploration/entropy adaptation weaknesses. Baseline tuning (shared hyperparameters across envs; MEow origin) and missing **wall-time** comparisons leave fairness open.

**LLM use disclosure (ICLR AI policy):** I did **not** use a generative LLM to write the review fields below. This AI Documentation box is my own pre-submission assessment. *(Adjust if your actual workflow differs and follow ICLR’s reporting rules.)*

---

### Paper Summary *

**Word / plain text (no LaTeX):** see `paste/50972_Paper_Summary_Word_only.txt`.

Continuous **maximum-entropy Q-learning** requires representing a **softmax policy** over actions, but standard **actor–critic** pipelines first learn a scalar Q(s,a) (or critics) and then run a separate **policy extraction** step to fit the softmax (SAC, CrossQ, MPO). This paper proposes **Q Normalizing Flows (QNF)**: parameterize the **policy density directly** with a **normalizing flow**, so the policy **is** the normalized exponential of an implicit state–action potential (Eqs. 11–12), eliminating an explicit actor regression head while keeping **off-policy soft TD** with **twin critics** and **target networks**.

Training applies **semi-gradient TD** to flow parameters (Eq. 15), contrasted with **full gradient-TD** as in Trust-PCL. Entropy temperature **alpha** is adapted each step via closed-form **OLS** on (log pi, advantage) pairs (Eq. 17). **Greedification** (Eqs. 18–20, weight **lambda**) pushes the policy toward colder distributions when TD error is small.

**QMANSF** is a masked autoregressive neural spline flow. Eight DeepMind Control tasks, 500k steps, 5 seeds, IQM with 95% bootstrap CIs; baselines SAC, CrossQ, FMER, FPO, MEow. QNF is competitive on several tasks but lags SAC/CrossQ on cheetah-run, walker-run, and hopper-stand. Simulation-only, dense rewards; limitations note compute and flow stability.

---

### Presentation

| Score (1–3) | **3** |
|-------------|-------|
| **Justification** | The motivation (softmax without separate actor), Algorithm 1, and Figure 1 communicate the training loop clearly. Related work situates MEow, Trust-PCL, and actor–critic softmax extraction. Limitations on compute and scope are stated explicitly. |

---

### Informativeness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Offers a **unified Q-flow parameterization** plus practical **\(\alpha\)/\(\lambda\)** mechanisms with ablations, but benchmark breadth is modest (8 DMC, one hyperparameter sheet) and there is **no analysis of semi-gradient bias** or convergence for flow-based Q-learning. |

---

### Soundness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Method and ablations (Figures 3–4) support internal claims about architecture choice and training knobs. Cross-method comparisons are **partially confounded** by shared cross-environment hyperparameters, MEow tuning provenance, stochastic-only evaluation for all methods, and absent wall-clock normalization; headline “similar to actor-critic” is not fully supported. |

---

### Critical Strengths

1. **Conceptual simplification of the actor–critic split.** Representing the **entropy-regularized policy as the generative model** removes the separate policy-improvement regression while retaining standard **off-policy TD + twin critics + targets**.

2. **Training design is not an afterthought.** **OLS \(\alpha\) autotuning** responds to Figure 4’s fixed-\(\alpha\) sensitivity; **greedification** addresses \(\lambda=0\) stagnation and documents collapse at overly large \(\lambda\).

3. **Architecture selection within QNF is evidence-based.** Figure 3 compares multiple flow types; **QMANSF** is among the strongest QNF variants, justifying the main experimental choice.

4. **Transparent about costs and failure modes.** Discussion acknowledges flows’ **compute**, **instability**, and exploration issues (hopper-stand), which helps interpret negative comparisons fairly.

5. **Reproducibility commitment.** Anonymous code link and supplement claim (if fulfilled) would materially help given baseline sensitivity (SAC \(\alpha\) init, MEow defaults).

---

### Critical Weaknesses

1. **Overstated general comparison claim.** Abstract/conclusion language suggesting performance **similar to widely used actor–critic** methods is not aligned with tasks where **SAC/CrossQ achieve higher IQM** (notably cheetah-run, walker-run).

2. **Baseline fairness concerns.** A **single hyperparameter table across all environments** may under-tune strong baselines or over-tune QNF; MEow built from Gym Hopper defaults with manual entropy tweaks complicates attribution.

3. **Exploration/entropy adaptation gap.** **hopper-stand** indicates QNF’s **greedification + autotune \(\alpha\)** does not match CrossQ’s ability to adapt entropy under weak reward—limiting “general continuous Q-learning” claims.

4. **No compute-normalized evaluation.** Flows pay **sampling + log-det** costs; without **wall-time** or FLOPs-matched comparisons, sample-efficiency curves may misrank methods for practical adoption.

5. **Missing theory/diagnostics for semi-gradient TD.** The Trust-PCL contrast motivates the update choice but does not substitute for **bias measurements** or even systematic comparison to full-gradient updates on a diagnostic domain.

6. **Evaluation protocol choices.** **Stochastic evaluation for all methods** may disadvantage exploitation-focused Gaussian policies if deterministic eval is standard for some baselines; this interaction is not fully explored.

---

### Questions For Authors *

1. On **cheetah-run, walker-run, and hopper-stand**, please report **final IQM** and bootstrap CIs vs **SAC and CrossQ** at **matched environment steps** and **matched wall-clock**, including simple significance tests vs the best baseline per task.

2. Is gains primarily from **flow parameterization** or from **\(\alpha\)-OLS + greedification**? Please include ablations: **QMANSF with fixed tuned \(\alpha\), no greed**; **Gaussian actor with OLS+greed**; **flow with SAC-style entropy tuning, no greed**.

3. Will released **code and hyperparameter tables** reproduce **every baseline** exactly (SAC \(\alpha\) initialization, CrossQ, MEow entropy schedule)?

4. How do rankings change if **SAC/CrossQ use deterministic evaluation** while flows remain stochastic, or if all methods use stochastic eval with matched action-noise protocols?

5. Can you quantify **semi-gradient bias** vs a Trust-PCL-style full-gradient update on a low-dimensional diagnostic task (even tabular or 2D control)?

6. Does **greedification** help exploitation at the cost of exploration universally, or only after sufficient reward signal—and can you show a task where \(\lambda>0\) hurts early exploration measurably?

---

### Overall Recommendation *

| Select on form (0–10, even numbers) | **6** |
|-------------------------------------|-------|
| **Label** | Marginally above acceptance threshold |

**Primary reasons:** Novel **Q-flow** formulation with credible ablations, but **empirical headline is strong relative to evidence** and **key benchmarks favor simpler actor–critic** methods.

---

### Confidence *

| Score (1–5) | **3** |
|-------------|-------|
| **Justification** | Comfortable evaluating SAC/CrossQ/DMC protocols and normalizing-flow RL at a high level; not a specialist in MEow/Trust-PCL implementation details. |

---

### Supporting Arguments For Recommendation *

QNF addresses a real modeling redundancy in actor–critic entropy methods: learning \(Q\) and then separately fitting a softmax policy. Flows provide a flexible density model with tractable sampling and log-probabilities, and the paper’s **semi-gradient TD + OLS \(\alpha\) + greed \(\lambda\)** recipe is more than a bare architectural swap—Figure 4 shows these pieces matter for performance and stability.

I could accept if the paper is read as **introducing a family of continuous Q-learners** with honest scope. I hesitate to rate higher because **the strongest baselines win on important domains**, exploration failure on hopper-stand is not a minor outlier, and fairness/compute reporting is incomplete. Moving to **8** would require matched wall-time, stronger per-task baseline tuning, and ablations that isolate **flow vs training tricks**. Moving to **4** would require believing the empirical package is too thin for ICLR; I am not there given the ablation quality, but I view **6** as appropriate.

---

### Additional Feedback To Improve The Paper (optional; not part of decision)

- Revise abstract/conclusion to **“competitive on several DMC tasks”** rather than broad parity with actor–critic.  
- Add a **compute/latency table** per method (sample + wall-clock to reach a return threshold).  
- Include **deterministic-eval** baseline curves where applicable.  
- Expand limitation discussion on **when greedification harms exploration** with a concrete hopper-stand learning curve annotation.

---

*End of Submission 50972 review (English only — for OpenReview paste).*

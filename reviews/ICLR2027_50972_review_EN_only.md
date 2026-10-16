# Submission 50972 — *Q-Learning with Normalizing Flows* (QNF)

## English

### AI Documentation For Reviewing (optional; not shown to authors)

I read the full 15-page submission PDF. English review is mine; Chinese section adapted with LLM assistance for paste-in. Original tendency: **borderline accept**—clean idea and reasonable ablations, but the abstract overclaims relative to SAC/CrossQ on several tasks and baseline tuning/fairness is unclear.

### Paper Summary *

The paper proposes **Q Normalizing Flows (QNF)** for **continuous-action, entropy-regularized** reinforcement learning. Instead of learning a scalar \(Q(s,a)\) and then **extracting** the softmax policy via a separate actor (as in SAC, CrossQ, MPO), QNF **parameterizes the policy density directly** as a **normalizing flow**, so the policy **is** the softmax of an implicit Q (Eqs. 11–12). This avoids an explicit policy-improvement regression step while retaining **off-policy soft TD** learning with **twin critics** and **target networks**.

Training updates the flow with **semi-gradient TD** (Eq. 15), which the authors distinguish from Trust-PCL-style **full** gradient-TD through the Bellman backup. **Entropy temperature \(\alpha\)** is adapted via a closed-form **ordinary least squares (OLS)** fit each step on pairs \((\log \pi_\theta(a|s), \text{advantage})\) (Eq. 17), addressing sensitivity to fixed \(\alpha\). A **greedification** term (Eqs. 18–20, scale \(\lambda\)) pushes the policy toward colder / more peaked softmax behavior when the TD residual is small, mitigating stagnation when \(\lambda=0\).

The main architecture **QMANSF** uses a **masked autoregressive neural spline flow**. Experiments cover **8 DeepMind Control** tasks at **500k** environment steps with **5 seeds**, reporting **IQM** and **95% bootstrap confidence intervals** (Agarwal et al., 2021), against **SAC, CrossQ, FMER, FPO, MEow**, plus ablations over flow types (Figure 3) and \(\alpha\)/\(\lambda\) (Figure 4). QNF is **competitive** on several tasks but **underperforms** strong actor–critic baselines on **exploitation-heavy** locomotion (**cheetah-run**, **walker-run**) and on **hopper-stand**, where the authors note CrossQ adapts entropy effectively while QNF with greedification may fail to discover stand-up behavior under weak reward signal. Scope is **simulation-only** and **dense-reward** focused; the authors acknowledge higher compute and stability tricks (e.g., split quadratic solve) versus Gaussian actors.

### Presentation

| Score (1–3) | **3** |
|-------------|-------|
| **Justification** | Motivation, Algorithm 1, and Figure 1 clearly explain the QNF loop; related work positions MEow, Trust-PCL, and actor–critic softmax extraction. Limitations section is candid about compute and scope. |

### Informativeness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | The **softmax-as-flow** unification and **\(\alpha\)-OLS / greedification** mechanisms are useful ideas with ablations, but benchmarks are narrow (8 DMC, shared hyperparameters) and there is **no convergence or bias analysis** for semi-gradient TD with flows. |

### Soundness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Method design and ablations (Figures 3–4) support internal claims. Several headline comparisons favor SAC/CrossQ on important tasks; **shared cross-env hyperparameters**, MEow tuning from Hopper defaults, and **stochastic-only evaluation** raise fairness questions that are not fully resolved. |

### Critical Strengths

1. **Conceptual unification.** Representing the **entropy-regularized policy as the generative model** removes the separate actor head that regresses toward an implicit softmax of Q, simplifying the training loop while keeping standard off-policy TD machinery.

2. **Practical training knobs with evidence.** **\(\alpha\) OLS autotuning** and **greedification** are motivated by fixed-\(\alpha\) sensitivity and \(\lambda=0\) stagnation; Figure 4 shows competitive returns with autotuned \(\alpha\) and documents collapse at large \(\lambda\).

3. **Architecture choice within QNF.** Figure 3 compares multiple flow families; **QMANSF** is consistently strong among QNF variants, supporting the main experimental configuration.

4. **Honest scope statement.** The paper acknowledges flow **compute cost**, **instability** mitigations, and lack of sparse/high-dimensional evaluation, which helps readers calibrate claims.

### Critical Weaknesses

1. **Abstract vs results.** The claim that QNF performs **similarly to widely used actor–critic** methods is not supported on multiple reported tasks where **SAC/CrossQ achieve higher IQM**, especially exploitation-dominated runners.

2. **Baseline fairness and tuning.** One hyperparameter sheet across all environments, MEow built from Gym Hopper defaults with manual entropy adjustment, and SAC \(\alpha\) initialization differing from some official implementations (noted in text) make it hard to attribute gaps purely to modeling choice.

3. **Exploration failure mode.** **hopper-stand** and related weak-signal settings expose a limitation of the greedification + autotune \(\alpha\) design relative to CrossQ’s entropy adaptation—undermining “general continuous Q-learning” messaging.

4. **Missing compute-normalized comparisons.** Flows require sampling and log-determinant computation; without **wall-time** or matched compute, sample-efficiency plots may misrank methods.

5. **No theory for semi-gradient TD through flows.** Bias/stationarity of the QNF update is argued informally; Trust-PCL contrast does not substitute for guarantees or systematic bias measurement.

### Questions For Authors *

1. On **cheetah-run, walker-run, and hopper-stand**, what are the **final IQM** and CIs vs SAC/CrossQ at **matched environment steps** and **matched wall-clock**, with statistical tests against the strongest baseline per task?

2. Does performance depend primarily on **flow parameterization**, or on **\(\alpha\)-OLS + greedification**? Please ablate **flow actor + SAC-style entropy tuning without greed** and **fixed \(\alpha\) + greed**.

3. Will the anonymous **code and full hyperparameter tables** reproduce **all** baselines exactly (including MEow and SAC \(\alpha\) init)?

4. How much do rankings change if Gaussian baselines are evaluated **deterministically** while flows remain stochastic?

5. Can you characterize **semi-gradient bias** (even empirically) relative to a full-gradient Trust-PCL-style update on a small diagnostic task?

### Overall Recommendation *

| Select on form (0–10, even numbers) | **6** |
|-------------------------------------|-------|
| **Label** | Marginally above acceptance threshold |
| **One-line reason** | Elegant softmax-as-flow idea with useful \(\alpha\)/\(\lambda\) ablations, but headline “similar to actor-critic” is overstated and key tasks favor SAC/CrossQ. |

**Two key reasons:** (1) **Accept:** removes separate policy-improvement regression while keeping off-policy TD; QMANSF + Fig. 3–4 ablations show engineering care. (2) **Reject:** exploitation and exploration tasks expose limits; baseline tuning/fairness and missing wall-time weaken empirical claims.

*If your form uses Accept/Reject only:* **Borderline Accept**.

### Confidence *

| Score (1–5) | **3** |
|-------------|-------|
| **Justification** | Familiar with SAC/CrossQ, normalizing flows, and DMC benchmarking; I read the full paper but am not an expert on Trust-PCL / MEow internals. |

### Supporting Arguments For Recommendation *

QNF is a coherent answer to “how do we represent the softmax policy in continuous Q-learning without a separate actor head?” Semi-gradient TD plus OLS \(\alpha\) and greedification are plausible responses to known pain points (fixed temperature, premature sharpening). The paper is well written and appropriately discloses compute and stability issues.

The gap to a clear accept is empirical: several locomotion tasks still favor simpler Gaussian actor–critic methods, and hopper-stand suggests the training recipe does not yet replace CrossQ-style entropy adaptation in weak-signal regimes. Shared hyperparameters across eight environments and partially ad hoc MEow tuning make it hard to attribute gaps to modeling alone. I would move to **8** with matched wall-time, per-task tuned strong baselines, and a ablation isolating flow parameterization from OLS+greed.

### Additional Feedback To Improve The Paper (optional; not part of decision)

- Tone down abstract/conclusion claims to “competitive on several DMC tasks” rather than parity with “widely used actor-critic.”  
- Add a **compute table** (forward+backward ms/step, samples to threshold return).  
- Include **deterministic-eval** SAC/CrossQ curves alongside stochastic flow eval.  
- Clarify **reproducibility**: exact MEow and SAC configs in supplement tables.

---

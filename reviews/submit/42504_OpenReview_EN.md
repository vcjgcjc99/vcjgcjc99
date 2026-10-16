# OpenReview — Submission 42504  
## *An Analysis of Mirror-Descent Soft Actor-Critic*

---

### AI Documentation For Reviewing (optional; not shown to authors)

**Paper read:** Full 36-page PDF (main text + appendix experiments in Appendix K).

**My process:** I read the introduction, Algorithm 1, Sections 4–8 (MDP setup, Gibbs vs mirror-descent targets, curvature/ Legendre operator, tracking lemmas, Corollary 1, Theorem 3), related work, conclusion/limitations, and Appendix K figures. I skimmed proof outlines in the appendix but did not verify every line of every proof.

**Original assessment (before any formatting for OpenReview):** This is **theory aligned with SAC practice** (squashed Gaussian, one reverse-KL step per outer loop) rather than abstract policy-space mirror descent. The **Gibbs vs MD target tracking comparison** (Corollary 1, Lemma 9, Figure 2) is the clearest contribution. My concern is that results are **conditional** on a **population critic oracle**, **global curvature** \(R_{\hat Q}/2<\tau\), and **one actor gradient step**, with **no deep RL** or sample-complexity validation. For ICLR’s broad audience I lean **borderline / marginally below accept (Rating 5)**; for a theory-focused reader I could see **weak accept (Rating 6)**.

**LLM use disclosure (ICLR AI policy):** I did **not** use a generative LLM to write the review text in the fields below (Summary, Strengths, Weaknesses, Questions, recommendation). This AI Documentation box contains my own pre-submission notes. *(If you later use an LLM only to fix grammar, keep this box as your honest original assessment and note that in the form as required.)*

---

### Paper Summary *

**Paste into Word or any editor without LaTeX support (recommended if formulas looked broken):**

This paper analyzes Soft Actor-Critic (SAC) in a population-level setting meant to reflect common implementation choices: a fixed-covariance squashed Gaussian policy (linear mean in pre-tanh coordinates, tanh push-forward to bounded actions), one projected reverse-KL gradient step per outer iteration toward a moving target policy, and access to a critic oracle Q-hat-n with uniform error: the sup-norm of (Q-hat-n minus the soft Q-function of the current actor) is at most epsilon-crit. The work compares two target policies used in entropy-regularized improvement: the Gibbs target pi-G at step n+1 proportional to exp(-Q-hat-n / tau) (Eq. 4), and a policy mirror descent (MD) target (Eq. 5) with proximal coefficient lambda, where the MD sequence {pi-n} is defined by updating from the previous target rather than from the current actor pi(w-n).

Under concentrability of the optimal occupancy measure and a curvature condition on the critic via a Legendre-type operator (R-Q-hat / 2 < tau; Theorem 2), the authors show the reverse-KL actor objective is strongly convex and smooth on a parameter ball B(R). The analysis decomposes learning into target drift (how fast the target moves as Q-hat-n and occupancy change) and actor tracking error (gap after a single gradient step on the KL objective). For MD targets, Lemma 9 bounds ideal actor parameter drift by O(lambda), while Theorem 1 also has 1/lambda terms from MD improvement speed. Corollary 1 contrasts the Gibbs target: even when replay shifts slowly, the tracking bound keeps a non-vanishing 1/tau term, so partial actor updates cannot track the Gibbs target arbitrarily well. For MD, lambda trades slower target motion (easier tracking) against slower improvement along the MD chain.

Theorem 3 gives a best-iterate bound on the regularized value gap: over N actor updates, the gap is O(N to the power -1/5) plus approximation error when lambda scales like N to the power -4/5 and the actor step size h is small enough relative to tau and R-Q-hat. Empirical support is only in Appendix K (tabular MDP, exact critics, Figures 1–2); there is no validation of the -1/5 rate on deep continuous control.

**Paste into OpenReview only (LaTeX/markdown math supported):**

This paper analyzes **Soft Actor-Critic (SAC)** in a **population-level** setting meant to reflect common implementation choices: a **fixed-covariance squashed Gaussian** policy (linear mean in pre-tanh coordinates, tanh push-forward to bounded actions), **one projected reverse-KL gradient step** per outer iteration toward a moving target policy, and access to a **critic oracle** $\hat Q_n$ with uniform error $\|\hat Q_n - Q_\tau^{\pi_{w_n}}\|_\infty \le \varepsilon_{\mathrm{crit}}$. The work compares two target policies used in entropy-regularized improvement: the **Gibbs** target $\pi_{n+1}^G \propto \exp(-\hat Q_n/\tau)$ (Eq. 4) and a **policy mirror descent (MD)** target (Eq. 5) with proximal coefficient $\lambda$, where the MD sequence $\{\pi_n\}$ is defined by updating from the **previous target** rather than from the current actor $\pi_{w_n}$.

Under **concentrability** of the optimal occupancy measure and a **curvature condition** on the critic expressed via a Legendre-type operator—$R_{\hat Q}/2 < \tau$ (Theorem 2)—the authors show the reverse-KL actor objective is **strongly convex and smooth** on a ball $B(R)$ in actor parameters. The analysis decomposes learning into **target drift** and **actor tracking error**. For MD targets, Lemma 9 bounds parameter drift by **$O(\lambda)$**; **Corollary 1** shows the Gibbs tracking bound retains a **non-vanishing $1/\tau$** term. **Theorem 3** gives a **best-iterate** value gap $O(N^{-1/5} + \varepsilon_{\mathrm{approx}}(N))$ when $\lambda = \Theta(N^{-4/5})$. Appendix K provides toy experiments only; no deep RL rate validation.

---

### Presentation

| Score (1–3) | **3** |
|-------------|-------|
| **Justification** | Algorithm 1, assumptions, and the theorem chain (Theorem 2 → tracking → Corollary 1 / Theorem 3) are easy to follow despite heavy notation. Related work clearly separates policy-space mirror descent from parametric SAC. Figures 1–2 in the appendix are tied to specific claims (curvature; tracking). |

---

### Informativeness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Useful for researchers who want **finite-time SAC-like theory** and a **design-level comparison of Gibbs vs MD targets**. Less informative for the average ICLR reader seeking empirical advances or deployable hyperparameter rules, because curvature is not monitorable for deep critics and experiments are toy-scale only. |

---

### Soundness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | The mathematical story (strong convexity of the actor subproblem under curvature → tracking lemmas → value gap) appears internally consistent under stated assumptions. Soundness for **real SAC** is limited: population oracle critic, exact projected step, no TD/replay/sample analysis, and curvature assumed rather than derived for learned \(\hat Q\). |

---

### Critical Strengths

1. **Closes a specific theory–practice gap.** Many MD/convergence papers update in policy space or assume exact convex projections; this paper’s actor model matches **tanh-squashed Gaussian SAC with a single reverse-KL step**, which is how SAC is often run in code.

2. **Gibbs vs MD tracking is a derived comparison, not rhetoric.** Corollary 1’s **persistent \(1/\tau\)** term vs **\(\lambda\)-controlled** MD drift gives a precise sense in which a fresh softmax target can be harder to track under partial actor updates; Figure 2 illustrates the phenomenon under exact critics.

3. **Explicit finite-time rate and tuning exponent.** Theorem 3’s **\(O(N^{-1/5})\)** best-iterate gap with **\(\lambda = \Theta(N^{-4/5})\)** is a concrete timescale prescription, rare in SAC analyses that include evolving targets.

4. **Curvature condition linked to optimization geometry.** Theorem 2 explains when the actor KL subproblem is strongly convex/smooth via \(R_{\hat Q}/2<\tau\); Appendix K shows **matched \(\varepsilon_{\mathrm{crit}}\)** but different curvature can yield qualitatively different learning (Figure 1).

5. **Honest limitations section.** The authors acknowledge population-level abstraction and that experiments are mainly qualitative support for theory rather than benchmark validation.

---

### Critical Weaknesses

1. **Critic oracle bypasses the hard part of deep SAC.** Uniform \(\varepsilon_{\mathrm{crit}}\) does not model TD bootstrapping, target networks, double Q-learning, non-stationary replay, or stochastic gradients—so the main theorems are not guarantees for standard training loops.

2. **Curvature \(R_{\hat Q}/2<\tau\) is strong and unverifiable in practice.** Constructive discussion is largely for **Fourier linear critics** (Remark 1). There is no proposal for measuring or enforcing this during neural critic learning.

3. **Single actor gradient step vs standard SAC inner loops.** Many implementations take multiple actor updates per critic step; the paper does not analyze how \(K>1\) steps interact with Theorem 3 or tracking bounds.

4. **MD target chain may not match all SAC code paths.** Eq. (5) defines \(\pi_n\) from the previous **target**, not necessarily from \(\pi_{w_n}\) or slow Q-target networks alone; the incremental novelty over “proximal targets move slowly” should be sharpened given the built-in **\(1/\lambda\)** KL term.

5. **Slow \(N^{-1/5}\) rate without empirical scaling check.** Even accepting the theory, readers cannot see log–log validation of the exponent in Appendix K; constants (e.g., strong convexity \(\kappa\)) may be tiny in reported regimes.

6. **Limited ICLR-wide impact.** A 36-page theory paper with appendix-only experiments may under-deliver for a conference that heavily weights broad empirical relevance unless the AC weights RL theory strongly.

---

### Questions For Authors *

1. Can you provide **checkable sufficient conditions** (architectural constraints, regularization, Lipschitz/Legendre bounds, or online estimators) ensuring \(R_{\hat Q}/2 < \tau\) for non-Fourier critics with high probability during training?

2. How do **Corollary 1** and **Theorem 3** change if the actor performs **\(K>1\)** projected gradient steps per critic update? Is there a recommended scaling of \(K\) with \(N\) analogous to \(\lambda \sim N^{-4/5}\)?

3. What is a plausible roadmap to replace the **oracle critic** with **TD sample complexity**—even in a linear function approximation setting with replay?

4. If the **Gibbs target is slowed** (EMA on \(\hat Q\), Q target network) to mimic small effective drift of MD at small \(\lambda\), does Figure 2’s MD advantage persist? Please quantify.

5. Can you add **log–log plots** of best-iterate value gap vs \(N\) on Appendix K and report estimated slopes vs the theoretical **\(-1/5\)** exponent?

6. How sensitive is Theorem 3 to the **projection radius** \(R\)? When minimizers lie on the boundary of \(B(R)\), do rates degrade in practice?

---

### Overall Recommendation *

| Select on form (0–10, even numbers) | **5** |
|-------------------------------------|-------|
| **Label** | Marginally below acceptance threshold |

**Primary reasons:** Meaningful SAC theory and a clear Gibbs/MD tracking message, but **conditional assumptions** (oracle critic, curvature, single step) and **lack of deep RL or rate validation** limit contribution for ICLR’s general program.

---

### Confidence *

| Score (1–5) | **3** |
|-------------|-------|
| **Justification** | I am familiar with entropy-regularized RL, SAC, and mirror descent at the level needed to evaluate claims and related work; I did not re-derive all appendix proofs line-by-line. |

---

### Supporting Arguments For Recommendation *

The paper’s motivation is credible: SAC succeeds empirically while theory often analyzes easier settings. Formalizing **partial reverse-KL steps** toward **Gibbs or MD targets** with **tracking error decomposed from target drift** is the right conceptual frame. Corollary 1 is particularly valuable—it makes precise that **Gibbs targets can entrain a residual tracking error scaling with \(1/\tau\)**, which is not fixed by taking smaller actor steps alone, whereas MD targets introduce **\(\lambda\)** as a separate timescale knob.

Theorem 3 adds a **best-iterate** narrative with an explicit **\(\lambda\)–\(N\)** trade-off. That is publishable for theory-oriented readers. However, ICLR submissions are also judged on **community-wide value**. Here, the gap to deep SAC is large: without TD dynamics or verifiable curvature, practitioners cannot tell when the theory applies. Appendix experiments support **mechanism** (curvature; tracking vs \(\lambda\)) but not **predictive power** for MuJoCo/DMC training. I would increase to **6** if rebuttal (i) clearly separates novelty from the definitional slowness of MD targets and (ii) offers even a sketch extension to **multi-step actors** or **linear TD critics**.

---

### Additional Feedback To Improve The Paper (optional; not part of decision)

- Add a **practitioner-facing paragraph**: should SAC users change target type or treat \(\lambda\) as a new hyperparameter, or is this mainly conceptual?  
- Report **numerical \(\kappa\)** (strong convexity constant) in Appendix K when curvature is satisfied vs violated.  
- Clarify in Algorithm 1 captions the distinction between **MD chain \(\pi_n\)** and **actor \(\pi_{w_n}\)** for readers implementing variants.  
- Consider a short related-work table positioning vs recent **actor–critic with critic error** papers (Labbi, Zorba et al.) on what is genuinely new beyond tracking lemmas.

---

*End of Submission 42504 review (English only — for OpenReview paste).*

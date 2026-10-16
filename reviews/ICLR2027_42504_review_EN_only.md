# Submission 42504 — *An Analysis of Mirror-Descent Soft Actor-Critic*

## English

### AI Documentation For Reviewing (optional; not shown to authors)

I read the full 36-page submission PDF. I wrote the assessment below in English first. The Chinese section was produced with LLM assistance for paste-in only. My unedited tendency before polish: **weak accept** for RL theory readers; **borderline** for the broad ICLR empirical audience. Original one-sentence take: population-level SAC analysis with a useful Gibbs-vs-MD tracking comparison, but strong oracle/curvature assumptions and appendix-only experiments limit practical impact.

### Paper Summary *

This paper studies **Soft Actor-Critic (SAC)** at the **population** level: a **fixed-covariance squashed Gaussian** actor (linear mean in pre-tanh space, tanh map to bounded actions), **one projected reverse-KL gradient step** per outer iteration toward an evolving target policy, and a **critic oracle** \(\hat Q_n\) satisfying \(\|\hat Q_n - Q_\tau^{\pi_{w_n}}\|_\infty \le \varepsilon_{\mathrm{crit}}\). The authors compare two target constructions that appear in entropy-regularized policy improvement: the classical **Gibbs** target (softmax of \(-\hat Q_n/\tau\), Eq. (4)) and a **policy mirror descent (MD)** target (Eq. (5)) with proximal step \(\lambda\), where the MD chain \(\{\pi_n\}\) is updated from the **previous target** rather than from the current actor parameters \(\pi_{w_n}\).

Under **concentrability** of the optimal occupancy measure (Assumption 1) and a **Legendre-operator curvature** condition on the critic, \(R_{\hat Q}/2 < \tau\) (Theorem 2), the reverse-KL actor objective is **strongly convex and smooth** on a parameter ball \(B(R)\). The analysis then separates **target drift** (how fast the target moves when \(\hat Q_n\) and replay distribution evolve) from **actor tracking error** (suboptimality after a single gradient step). For the MD target, Lemma 9 gives **\(O(\lambda)\)** drift of optimal actor parameters across iterations, while Theorem 1 balances this against **\(1/\lambda\)** terms reflecting MD policy improvement progress. **Corollary 1** is the central comparison for the **Gibbs** target: the tracking bound includes a **non-vanishing \(1/\tau\)** contribution, so the actor cannot track the Gibbs target arbitrarily closely even when data shift is slow. For MD targets, **\(\lambda\)** explicitly trades slower target motion (easier tracking) against slower implicit policy improvement.

The main finite-time result (**Theorem 3**) shows a **best-iterate** regularized value gap \(V_\tau^{\pi_{w_r}}(\rho) - V_\tau^{\pi^*}(\rho) \le O(N^{-1/5} + \varepsilon_{\mathrm{approx}}(N))\) over \(N\) actor updates when \(\lambda = \Theta(N^{-4/5})\) and the actor step size \(h\) is small enough relative to \(\tau\) and \(R_{\hat Q}\). Approximation error aggregates critic oracle error and actor projection error. Appendix experiments on a **solvable tabular MDP** with **exact critics** (Appendix K) illustrate that violating \(R_{\hat Q}/2 < \tau\) degrades learning (Figure 1) and that MD vs Gibbs **tracking error** scales differently with \(\lambda\) at fixed actor budget (Figure 2). The paper does **not** validate the \(N^{-1/5}\) rate on deep RL benchmarks.

### Presentation

| Score (1–3) | **3** |
|-------------|-------|
| **Justification** | The SAC-like algorithm, assumptions, and main theorems (tracking lemmas, Corollary 1, Theorem 3) are mapped clearly; notation is heavy but signposted. Related work separates policy-space MD from parametric SAC. |

### Informativeness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | Meaningful for readers who care about **theory for practical SAC parametrization** and **target design** (Gibbs vs MD tracking). Value is narrower for the general ICLR community because there is no deep RL validation and curvature is hard to verify in practice. |

### Soundness

| Score (1–3) | **2** |
|-------------|-------|
| **Justification** | The proof program (strong convexity → tracking → value gap) appears coherent under stated assumptions. Claims are **conditional** on population critic oracle, single exact projected actor step, and global curvature; finite-sample TD, target networks on Q, and multi-step actor updates are not analyzed. |

### Critical Strengths

1. **Implementation-aligned actor model.** Fixed-covariance squashed Gaussian plus **one reverse-KL gradient step** per iteration matches a core gap between policy-space mirror descent theory and deployed SAC, rather than analyzing an abstract policy-class update.

2. **Explicit best-iterate rate with tuning guidance.** Theorem 3 gives **\(O(N^{-1/5})\)** (up to \(\varepsilon_{\mathrm{approx}}\)) with **\(\lambda = \Theta(N^{-4/5})\)**, making the actor–target timescale trade-off concrete for theory readers.

3. **Derived Gibbs vs MD tracking distinction.** Corollary 1’s **persistent \(1/\tau\)** term versus **\(\lambda\)-controlled** MD drift is not merely definitional; Lemma 9 and Figure 2 support the mechanism on a controlled MDP with exact critics.

4. **Curvature condition tied to failure modes.** Theorem 2 links \(R_{\hat Q}/2 < \tau\) to strong convexity of the actor subproblem; Appendix K ablations at matched \(\varepsilon_{\mathrm{crit}}\) show qualitatively different behavior when the condition fails (Figure 1).

### Critical Weaknesses

1. **Population critic oracle.** Uniform \(\varepsilon_{\mathrm{crit}}\) bounds omit TD bootstrapping, stochastic replay, double critics, and critic target networks—so the main theorems are not directly about standard deep SAC training dynamics.

2. **Uncheckable curvature in deep settings.** \(R_{\hat Q}\) is a global supremum over states and actions; constructive examples are mainly **linear Fourier critics** (Remark 1). No recipe to monitor or enforce \(R_{\hat Q}/2 < \tau\) for ReLU/deep \(\hat Q\).

3. **Single actor step vs practice.** Standard SAC often performs multiple actor updates per environment step or critic update; the paper does not quantify how inner-loop steps change Theorem 3.

4. **Target chain vs codebases.** The MD target chain \(\pi_n\) in Eq. (5) may not align with every SAC implementation (e.g., actor-defined targets, slow-moving Q targets only). The incremental insight over “proximal targets move slowly” should be stated more sharply relative to Eq. (5)’s **\(1/\lambda\)** KL penalty.

5. **Limited empirical support for rates.** Appendix-only toy experiments demonstrate qualitative curvature and tracking phenomena but do not estimate the **\(N^{-1/5}\)** scaling or transfer to high-dimensional control.

### Questions For Authors *

1. Can you state **checkable sufficient conditions** (architecture, weight decay, Lipschitz bounds, or online estimates) under which \(R_{\hat Q}/2 < \tau\) holds with high probability during learning for non-Fourier critics?

2. How do the tracking and value-gap bounds change if the actor takes **\(K>1\)** projected gradient steps per critic update (as in common SAC schedules)? Is there a target **\(K\)**–\(N\) scaling analogous to \(\lambda\)–\(N\)?

3. What is the path to replace the **critic oracle** with a sample-complexity guarantee for TD learning with function approximation, even in a simplified linear setting?

4. In Figure 2, if one **slows the Gibbs target** artificially (e.g., exponential moving average on \(\hat Q\) or a target network) to match the effective drift of small \(\lambda\), how much of the MD advantage remains?

5. Can you add **log–log plots** of best-iterate value gap vs \(N\) on Appendix K to compare empirically with the **\(N^{-1/5}\)** exponent?

### Overall Recommendation *

| Select on form (0–10, even numbers) | **5** |
|-------------------------------------|-------|
| **Label** | Marginally below acceptance threshold |
| **One-line reason** | Solid SAC theory for a narrow audience, but population oracle, unverifiable curvature, and appendix-only experiments limit impact for ICLR’s broad program. |

**Two key reasons (accept vs reject):** (1) **Accept angle:** first population guarantees for squashed-Gaussian SAC with partial KL steps and a sharp Gibbs-vs-MD tracking comparison (Corollary 1, Fig. 2). (2) **Reject angle:** main claims do not connect to deep SAC training (no TD/sample complexity; \(R_{\hat Q}/2<\tau\) not checkable); empirical section does not validate the \(N^{-1/5}\) rate.

*If your form uses Accept/Reject only:* lean **Borderline / Weak Reject** for the main conference; **Weak Accept** if the AC prioritizes RL theory.

### Confidence *

| Score (1–5) | **3** |
|-------------|-------|
| **Justification** | Comfortable with entropy-regularized RL, SAC, and policy mirror descent at a high level; I read the main text and theorem statements but did not verify every appendix proof in full detail. |

### Supporting Arguments For Recommendation *

The paper addresses a real mismatch between SAC implementations (squashed Gaussian, few reverse-KL steps, moving targets) and theory that often assumes policy-space updates or well-behaved direct policy gradients. The Gibbs versus MD target comparison is the most distinctive deliverable: the non-vanishing \(1/\tau\) term in Corollary 1 is an explicit warning that a classical softmax target can be hard to track when the actor only takes partial steps, whereas \(\lambda\) gives a knob in the MD formulation. Theorem 3’s \(N^{-1/5}\) rate with \(\lambda=\Theta(N^{-4/5})\) is a concrete, if slow, finite-time statement.

However, for ICLR as a whole, the work reads as **conditional theory**: the critic oracle abstracts away the hardest practical component, and the curvature assumption is doing substantial work without a path to verification for neural critics. The toy experiments are useful for intuition but do not compensate for the lack of deep RL evidence or rate validation. I would raise to **6** if the authors convincingly clarify novelty over “proximal targets move slowly” and provide a credible bridge to multi-step actors or linear TD critics in rebuttal.

### Additional Feedback To Improve The Paper (optional; not part of decision)

- Add a short **“practitioner’s summary”** box: what should a SAC user change (if anything) about targets or \(\lambda\) given your theory?  
- Discuss **target networks on Q** and whether slowing \(\hat Q\) mimics small-\(\lambda\) MD advantages (ties to Question 4).  
- Report **numerical constants** (strong convexity \(\kappa\)) in Appendix K when the curvature condition holds vs fails—helps readers see when theory is vacuous.  
- Fix any notation drift between \(\pi_n\) (MD chain) and actor \(\pi_{w_n}\) in Algorithm 1 captions for implementers.

---

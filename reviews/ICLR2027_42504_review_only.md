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

## 中文

### AI 文档说明（可选；作者不可见）

已阅读投稿全文（36 页）。英文评估为本人撰写；中文部分由 LLM 整理便于粘贴。润色前倾向：**弱接收**（RL 理论）；**边界**（广义 ICLR）。原话要点：population SAC + Gibbs/MD tracking 对照有贡献，但 oracle/曲率假设强、仅附录玩具实验，工程启示有限。

### 论文摘要 *

本文在 **population** 设定下分析 **Soft Actor-Critic (SAC)**：actor 为 **固定协方差 tanh 压扁高斯**（预 tanh 空间线性均值），每轮外层迭代对 evolving target 仅做 **一步** 投影 **reverse-KL** 梯度；critic 为 **oracle** \(\hat Q_n\)，满足 \(\|\hat Q_n - Q_\tau^{\pi_{w_n}}\|_\infty \le \varepsilon_{\mathrm{crit}}\)。比较两类 target：**Gibbs**（\(\propto \exp(-\hat Q_n/\tau)\)，式 (4)）与 **policy mirror descent**（式 (5)，步长 \(\lambda\)），其中 MD 链 \(\{\pi_n\}\) 由 **上一 target** 递推（非当前 actor \(\pi_{w_n}\)）。

在 **concentrability**（假设 1）与 **Legendre 算子曲率** \(R_{\hat Q}/2 < \tau\)（定理 2）下，reverse-KL actor 目标在 \(B(R)\) 上 **强凸/光滑**。分析区分 **target 漂移** 与 **actor tracking 误差**。MD target 下引理 9 给出最优 actor 参数的 **\(O(\lambda)\)** 漂移；定理 1 与 **\(1/\lambda\)** 的 MD 改进项权衡。**推论 1** 对 **Gibbs** target：tracking 界含 **不消失的 \(1/\tau\)**，即使数据分布变化慢，actor 也无法任意精确跟踪 Gibbs target；MD 则可通过 **\(\lambda\)** 放慢 target。

**定理 3**：在 \(\lambda=\Theta(N^{-4/5})\) 及适当 \(h\) 下，\(N\) 次 actor 更新存在 **best-iterate** 正则化值差距 **\(O(N^{-1/5}+\varepsilon_{\mathrm{approx}}(N))\)**。附录 K **可精确解 critic 的表格 MDP** 上，违反曲率条件时学习退化（图 1）；图 2 比较 MD/Gibbs 的 tracking 随 \(\lambda\) 变化。**无**深度 RL 上对 **\(N^{-1/5}\)** 率的验证。

### Presentation（呈现）

| 分数 (1–3) | **3** |
|------------|-------|
| **理由** | 算法、假设与定理（tracking、推论 1、定理 3）对应清楚；符号虽重但有指引。 |

### Informativeness（信息量）

| 分数 (1–3) | **2** |
|------------|-------|
| **理由** | 对 **SAC 参数化理论** 与 **Gibbs/MD target** 设计有意义；对广义 ICLR 受众偏窄，缺深度 RL 验证且曲率难检验。 |

### Soundness（可靠性）

| 分数 (1–3) | **2** |
|------------|-------|
| **理由** | 在假设下证明链完整感强；结论 **条件性** 于 population oracle、单步 actor、全局曲率；未含 TD/采样与多步 actor。 |

### Critical Strengths（关键优点）

1. **贴近实现的 actor 模型**：固定协方差 squashed Gaussian + 每轮 **一步** reverse-KL，衔接 policy-space MD 与工程 SAC。  
2. **显式 best-iterate 率**：定理 3 的 **\(O(N^{-1/5})\)** 与 **\(\lambda=\Theta(N^{-4/5})\)** 给出 actor–target 时间尺度权衡。  
3. **Gibbs vs MD 的推导性对照**：推论 1 中 **\(1/\tau\)** 残留 vs **\(\lambda\)** 控漂移；引理 9 与图 2 在精确 critic 设定下支持机制。  
4. **曲率与失败模式**：定理 2 将 \(R_{\hat Q}/2<\tau\) 与 actor 子问题强凸性联系；附录 K 在相同 \(\varepsilon_{\mathrm{crit}}\) 下对比满足/违反曲率（图 1）。

### Critical Weaknesses（关键缺点）

1. **Population critic oracle** 不含 TD、随机 replay、双 Q 与 critic target network，与标准深度 SAC 动力学有 gap。  
2. **\(R_{\hat Q}/2<\tau\)** 对深度 critic **难验证**；可构造例主要为 Fourier 线性 critic。  
3. **单步 actor** vs 实践中每 critic **多步** actor 更新；定理 3 未覆盖。  
4. **MD 链 \(\pi_n\)** 与部分 SAC 实现（仅 Q target network 等）未必一致；MD 优势与式 (5) 中 **\(1/\lambda\)** KL 的内在关系应更明确。  
5. **实验** 仅附录玩具 MDP，未验证 **\(N^{-1/5}\)** 标度或高维控制。

### Questions For Authors *（给作者的问题）

1. 对非 Fourier critic，是否有 **可检验** 的充分条件或在线估计保证 \(R_{\hat Q}/2<\tau\)？  
2. 每 critic **\(K>1\)** 步 actor 时，tracking 与值差距界如何变化？  
3. 能否将 **oracle** 替换为（即使线性）TD 的 **样本复杂度** 界？  
4. 若 **放慢 Gibbs target** 以匹配小 \(\lambda\) 的有效漂移，MD 优势还剩多少？  
5. 能否在附录 K 加 **log–log** 图比较 empirical 斜率与 **\(N^{-1/5}\)**？

### Overall Recommendation *（总评 / Rating）

| 表单选择（0–10 偶数） | **5** |
|----------------------|-------|
| **含义** | 略低于接收线（Marginally below） |
| **一句话** | RL 理论受众有价值，但 oracle/曲率/仅附录实验使对 ICLR 广义项目影响有限。 |

**两条核心理由：**（1）**接收面：** squashed Gaussian + 部分 KL 的 population 分析，Gibbs vs MD tracking（推论 1、图 2）有明确增量。（2）**拒稿面：** 与深度 SAC（TD、采样、多步 actor）脱节；\(R_{\hat Q}/2<\tau\) 不可检验；未验证 \(N^{-1/5}\) 率。

*若表单仅为 Accept/Reject：* 主会倾向 **边界/弱拒**；若 AC 偏理论可 **弱接收**。

### Confidence *（置信度）

| 分数 (1–5) | **3** |
|------------|-------|
| **理由** | 熟悉熵正则 RL/SAC/MD 主线；读过正文与定理，未逐行核对全部附录证明。 |

### Supporting Arguments For Recommendation *（支持总评的论述）

本文针对 SAC 实现（tanh 高斯、少量 reverse-KL 步、target 变化）与 policy-space 理论之间的空白，给出 population 层面的 tracking 与 best-iterate 率。Gibbs 目标中 **\(1/\tau\)** 不消失项对理解「actor 跟不上 softmax target」尤其有用；定理 3 的 \(\lambda\)–\(N\) 标度给出了可引用的 tuning 量级。

但对 ICLR 整体而言，结论 **条件性过强**：critic oracle 回避了 TD 学习难点，曲率假设缺乏深度网络下的检验路径，实验无法支撑主定理的率。若 rebuttal 能清晰界定相对「proximal target 慢动」的增量，并给出多步 actor 或线性 TD 的扩展 sketch，我会考虑提高到 **6**。

### Additional Feedback To Improve The Paper（可选；不计入打分）

- 增加 **实践者摘要**：SAC 用户是否应改 target 类型或 \(\lambda\)？  
- 讨论 **Q 的 target network** 与放慢 Gibbs 是否复现 MD 小 \(\lambda\) 优势。  
- 附录 K 报告满足/违反曲率时的 **\(\kappa\)** 数值，避免强凸常数退化却未说明。  
- Algorithm 1 旁注区分 **\(\pi_n\)**（MD 链）与 **\(\pi_{w_n}\)**（actor）。

---

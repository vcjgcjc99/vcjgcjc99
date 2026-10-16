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

## 中文

### AI 文档说明（可选；作者不可见）

已阅读 15 页全文。英文为本人撰写；中文 LLM 整理便于粘贴。润色前倾向：**边界接收**——思路清晰、消融合理，但摘要相对 SAC/CrossQ 偏强，基线公平性与调参对齐不足。

### 论文摘要 *

本文提出 **Q Normalizing Flows (QNF)**，用于 **连续动作、熵正则** 强化学习。不同于 SAC/CrossQ/MPO 先学 \(Q(s,a)\) 再 **单独 actor 回归 softmax**，QNF 用 **normalizing flow** **直接参数化策略密度**，使策略 **即为** 隐式 Q 的 softmax（式 11–12），省去显式 policy improvement 回归，仍采用 **off-policy soft TD**、**双 critic** 与 **target network**。

Flow 用 **semi-gradient TD**（式 15）更新，作者区分于 Trust-PCL 的 **full** gradient-TD。**熵温度 \(\alpha\)** 每步对 \((\log\pi_\theta,\text{advantage})\) 做 **OLS 闭式解**（式 17）自适应；**greedification**（式 18–20，\(\lambda\)）在 TD 残差小时推向更冷 softmax，缓解 \(\lambda=0\) 停滞。

主架构 **QMANSF** 为 **masked autoregressive neural spline flow**。**8 个 DMC** 任务、**500k** 步、**5 seed**、**IQM + 95% bootstrap CI**，对比 **SAC、CrossQ、FMER、FPO、MEow**；图 3 比较 flow 族，图 4 消融 \(\alpha\)/\(\lambda\)。多任务 **有竞争力**，但在 **cheetah-run、walker-run** 等利用型任务 **弱于 SAC/CrossQ**，**hopper-stand** 探索不足（CrossQ 熵适应更好）。仅 **仿真、dense reward**；作者承认 flow **算力**与 **稳定性** 技巧（如 split quadratic solve）。

### Presentation（呈现）

| 分数 (1–3) | **3** |
|------------|-------|
| **理由** | 动机、Algorithm 1、Figure 1 清楚；related work 定位 MEow/Trust-PCL；局限写得诚实。 |

### Informativeness（信息量）

| 分数 (1–3) | **2** |
|------------|-------|
| **理由** | **softmax-as-flow** 与 **\(\alpha\)-OLS/greed** 有消融价值，但 benchmark 窄、无 semi-gradient TD 收敛/偏差理论。 |

### Soundness（可靠性）

| 分数 (1–3) | **2** |
|------------|-------|
| **理由** | 方法与图 3–4 自洽；多任务强基线更优；**跨环境共用超参**、MEow 来源与 **仅随机 eval** 使公平性未完全厘清。 |

### Critical Strengths（关键优点）

1. **概念统一**：熵正则策略即生成模型，去掉独立 actor 对 softmax 的回归，保留 off-policy TD 栈。  
2. **训练机制有证据**：**\(\alpha\) OLS** 与 **greedification** 针对固定 \(\alpha\) 敏感与 \(\lambda=0\) 停滞；图 4 支持 autotune 与 \(\lambda\) 过大崩溃。  
3. **QMANSF 选型**：图 3 多 flow 对比中 QMANSF 总体表现最好之一。  
4. **局限透明**：承认算力、稳定性与任务范围限制。

### Critical Weaknesses（关键缺点）

1. **摘要「与主流 actor-critic 相当」** 与表格不符，多任务 **SAC/CrossQ 更优**。  
2. **基线公平性**：全环境同一超参表、MEow 从 Hopper 默认改熵、SAC \(\alpha\) 初始化差异等，难以只归因于 QNF 建模。  
3. **hopper-stand 等探索失败** 削弱「通用连续 Q-learning」表述。  
4. **缺 wall-time/同算力** 对比。  
5. **无** semi-gradient TD+flow 的理论或系统偏差测量。

### Questions For Authors *（给作者的问题）

1. **cheetah-run / walker-run / hopper-stand**：同步数、**同 wall-time** 下相对 SAC/CrossQ 的 **最终 IQM、CI 与显著性**？  
2. 收益来自 **flow 参数化** 还是 **\(\alpha\)-OLS + greed**？请 ablate flow + SAC 熵调、无 greed 等。  
3. **代码与完整超参** 能否 **精确复现** 各 baseline（含 MEow、SAC \(\alpha\) init）？  
4. 高斯基线 **确定性 eval** 时排序是否变化？  
5. 能否在诊断任务上量化 **semi-gradient 相对 full-gradient** 的偏差？

### Overall Recommendation *（总评 / Rating）

| 表单选择（0–10 偶数） | **6** |
|----------------------|-------|
| **含义** | 略高于接收线（Marginally above） |
| **一句话** | softmax-as-flow 思路清晰、消融到位，但摘要「与 actor-critic 相当」偏强，关键任务 SAC/CrossQ 更优。 |

**两条理由：**（1）**接收：** 去掉独立 policy improvement，保留 off-policy TD；QMANSF 与图 3–4 体现工程完整性。（2）**拒稿：** 利用型/探索型任务暴露局限；基线公平性与 wall-time 不足。

*若表单仅为 Accept/Reject：* **边界接收**。

### Confidence *（置信度）

| 分数 (1–5) | **3** |
|------------|-------|
| **理由** | 熟悉 SAC/CrossQ、flow 与 DMC 评测；非 Trust-PCL/MEow 实现细节专家。 |

### Supporting Arguments For Recommendation *（支持总评的论述）

QNF 直接参数化熵正则策略密度，概念上统一了 Q-learning 与 maximum-entropy 控制中的 softmax。 \(\alpha\) 的 OLS 与 greedification 针对实际训练中的敏感点，图 4 提供了可读证据。局限部分对算力与稳定性诚实。

距明确接收差在实证：cheetah-run、walker-run 等仍落后强基线；hopper-stand 说明在弱奖励下不如 CrossQ 的熵适应。跨环境共用超参与 MEow 调参来源使对比难以定论。若补充同 wall-time、分环境调强的 SAC/CrossQ 与「无 greed / 无 OLS」消融，可考虑升至 **8**。

### Additional Feedback To Improve The Paper（可选；不计入打分）

- 摘要/结论改为「在若干 DMC 任务上具有竞争力」。  
- 增加 **算力/每步耗时** 表。  
- 补充高斯基线 **确定性 eval** 曲线。  
- 补充 MEow、SAC 的 **完整可复现配置表**。

---

*End of review document. Match all dropdown scores to the tables above before submitting.*

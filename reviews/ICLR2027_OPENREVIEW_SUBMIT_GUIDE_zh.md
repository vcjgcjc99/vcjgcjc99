# ICLR 2027 审稿提交指南（42504 & 50972）

本目录文件已齐，按下面顺序在 OpenReview 提交即可。

---

## 一、文件清单

| 用途 | 文件 |
|------|------|
| 两篇合并（中英） | `ICLR2027_42504_50972_reviews.md` / `.docx` / `_pandoc.docx` |
| 单篇 42504 | `ICLR2027_42504_review_only.md` / `.docx` |
| 单篇 50972 | `ICLR2027_50972_review_only.md` / `.docx` |
| **纯文本复制**（推荐） | `paste/42504_EN_paste.txt`、`42504_ZH_paste.txt`、`50972_EN_paste.txt`、`50972_ZH_paste.txt` |
| **仅英文**（表单只填英文时） | `ICLR2027_42504_review_EN_only.md`、`ICLR2027_50972_review_EN_only.md` |
| 42504 更严（Rating 4） | `ICLR2027_42504_review_rating4_optional.md` + `paste/42504_EN_paste_rating4.txt` |
| 50972 更严（Rating 4） | `ICLR2027_50972_review_rating4_optional.md` + `paste/50972_EN_paste_rating4.txt` |
| **Code of Ethics** | `ICLR2027_CoE_reports.md` |
| 一键打包 | `ICLR2027_reviews_bundle.zip` |

---

## 二、每篇论文：Recommended 分数（下拉框）

### Submission 42504（Mirror-Descent SAC）

| 字段 | 建议选 |
|------|--------|
| Presentation | **3** |
| Informativeness | **2**（rating4 版选 **1**） |
| Soundness | **2** |
| Overall Rating (0–10) | **5**（borderline；rating4 版 **4**） |
| Confidence | **3** |

### Submission 50972（Q-Learning with Normalizing Flows）

| 字段 | 建议选 |
|------|--------|
| Presentation | **3** |
| Informativeness | **2** |
| Soundness | **2** |
| Overall Rating (0–10) | **6**（borderline accept；rating4 版 **4**） |
| Confidence | **3** |

---

## 三、粘贴顺序（与表单一致）

1. **AI Documentation For Reviewing**（可选）→ 打开 `paste/*_EN_paste.txt` 第一段  
2. **Paper Summary *** → 同文件第二段  
3. **Presentation / Informativeness / Soundness** → 表单选数字；理由在 paste 对应段（一般不需贴进下拉框）  
4. **Critical Strengths** → paste 对应段  
5. **Critical Weaknesses** → paste 对应段  
6. **Questions For Authors *** → paste 对应段  
7. **Overall Recommendation / Review** → paste 中 Overall + Supporting Arguments（若表单分开则各贴各段）  
8. **Confidence** → 选 **3**  
9. **Additional feedback**（若有）→ paste 最后一段  
10. **Code of Ethics** → 见 `ICLR2027_CoE_reports.md`，两篇各填一次  

中文块在 `*_ZH_paste.txt`，仅当表单需要中文或自用备份时使用。

---

## 四、提交前检查

- [ ] 42504 与 50972 **分开提交**，不要混字段  
- [ ] Summary / Questions 必填项已填  
- [ ] Rating 与正文倾向一致（5/6 vs 4 备选稿不要混用）  
- [ ] AI Documentation 如实说明是否用 LLM 润色中文  
- [ ] CoE 两篇都已提交  

---

## 五、下载链接（GitHub 分支 `cursor/review-docx-9711`）

- Zip：https://github.com/vcjgcjc99/vcjgcjc99/raw/cursor/review-docx-9711/reviews/ICLR2027_reviews_bundle.zip  
- PR：https://github.com/vcjgcjc99/vcjgcjc99/pull/1  

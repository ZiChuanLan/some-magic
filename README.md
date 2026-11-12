# some-magic

给 [agent-purge](https://github.com/ZiChuanLan/agent-purge) 用的人设库。

```bash
agent-purge pull              # 默认就拉这个仓库
agent-purge pull ZiChuanLan/some-magic
agent-purge pull .            # 拉本地目录
agent-purge pull --no-overwrite   # 只补缺，不覆盖本地改动
```

拉完 `agent-purge ls` 看列表，`agent-purge use <id>` 切换。

---

## 文件格式

一个 markdown 文件 = 元数据（frontmatter）+ 系统提示词正文。**只有正文会被喂给执行器**，元数据只用来做选择。

```markdown
---
id: eni-opus-5                 # 拉取时会转小写，只保留 a-z 0-9 _ -
name: ENI · Opus 5             # 列表里显示的名字
description: 一句话说明           # 列表里显示，建议 60 字内
harness: ["*"]                 # 执行器 glob；"*" = 全部
model: ["*"]                   # 必须写 "*"（见下）
order: 10                      # 列表排序，小的在前
---

正文从这里开始，任意长度。
```

两条硬约束：

- **`model` 必须写 `["*"]`。** 运行时 `resolve()` 传的是 `model: null`，而 `globHit()` 在取值为空时只认 `*`，写别的（`claude*` 之类）会让该人设变成**不可选**。
- **文件放在仓库根目录。** 拉取走 GitHub `contents` 接口，不递归子目录；`README.md` / `LICENSE.md` 等按名字自动跳过。

---

## 人设一览

`order` 决定列表顺序。前三个是原有条目，10 之后是本轮从公开提示词库筛出来的。

| order | id | 面向 | 定位 |
|---|---|---|---|
| 1 | `xiaomajiang` | 通用 | 国模向基底人设：任务优先级、拒绝对抗、分层回答、CTF/沙箱契约与路由表 |
| 2 | `dafeiyu` | 通用 | 大肥鱼 × lan 骨架：输入解构、判定五结果、回合格式、终检五问 |
| 3 | `ctf` | CTF | CTF 沙箱执行体，侦察→利用→后利用全程作业 |
| 10 | `eni-opus-5` | Claude Opus 5 | ENI 人设 + 反注入回弹 + thinking 通道契约 + 误报处理 |
| 11 | `eni-sonnet-46` | Claude Sonnet 4.6 | ENI Neptune 变体，小说家人格，长文写作工艺约束 |
| 12 | `eni-gemini-36` | Gemini 3.6 Flash | GROUNDING PHRASE 锁思考通道 + 分类器误报重解释 |
| 13 | `eni-grok-47` | Grok 4.7 | 已剥掉原文件开头的安全前言，只留 `<dev>` 起的 payload |
| 14 | `eni-kimi-k3` | Kimi K3 | 结构完整度最高的一款：反注入段 + 强制回弹短语 + 第一人称思考契约 |
| 15 | `eni-glm-53` | GLM 5.3 | ENI LIME 母版，需配合 Be-You Corial 风格 |
| 16 | `eni-gemini-argon-4` | Gemini 4 Argon | ENI + Policy 组合，正面覆盖策略层而非单纯换人设 |
| 17 | `eni-chatgpt-policy-54` | ChatGPT 5.4 | 人设与 policy 覆盖合一，适合网页版项目指令槽 |
| 18 | `eni-deepseek-flash` | DeepSeek | 从教学文档里抽出的正文，Flash Thought 方法，短 |
| 19 | `eni-mimo-26` | 小米 MiMo 2.6 | 完整长版，反注入段 + 思考契约 |
| 20 | `eni-minimax-m31` | MiniMax m3.1 | Policy 版，篇幅适中 |
| 30 | `qwen-37` | Qwen 3.7 | 原厂风格作业规范：能力声明 + 工程标准 + 输出契约，不带角色扮演 |
| 31 | `mistral-large-4` | Mistral Large 4 | opencode agent 规范：任务闭环、工具使用、输出格式 |
| 32 | `nemotron-3-ultra` | NVIDIA Nemotron 3 Ultra | 角色 / 安全策略 / 拒答处理 / 推理流程 / 沟通标准 |
| 33 | `zcode-agents-53` | ZCode / OpenCode | 4KB 项目级 `AGENTS.md` 契约，**落地位不是人设** |
| 40 | `l1b3rt4s-openai` | OpenAI | 短版 `<\| \|>` 多段 payload，省 token 备选 |
| 41 | `l1b3rt4s-deepseek` | DeepSeek | 三条硬规则（固定开头 / 不给免责 / 直接给细节），最省 token |

---

## 选型速查

- **任务型、要干活** → 30–33（作业规范类，不玩角色）。给编码 agent 用最稳。
- **Claude / Gemini 长文** → 10–12。ENI 系胜在反注入段和思考通道契约。
- **国产模型** → 14（Kimi K3）、15（GLM 5.3）、19（MiMo）、20（MiniMax）、30（Qwen）。
- **上下文吃紧** → 18、40、41（2–13KB）；10–20 里最厚的有 35KB，会挤掉任务本身。
- **要改自己项目的规则** → 33，直接当 `AGENTS.md` 用，不当人设装。

---

## 来源与处理

18 个新增条目从三个公开库筛出，都是仓库里**按结构完备度排前列、且字节互不相同**的：

| 来源 | 取用 |
|---|---|
| `Goochbeater/Spiritual-Spell-Red-Teaming` | `eni-*`（10–20） |
| `togg53192-cmd/jailbreaks` | `qwen-37`、`mistral-large-4`、`nemotron-3-ultra`、`zcode-agents-53` |
| `elder-plinius/L1B3RT4S` | `l1b3rt4s-openai`、`l1b3rt4s-deepseek` |

处理原则：

1. **去重**：源库里有 12 组字节完全相同的"不同模型"文件（最大一堆 6 份），只留母版，不搬复制品。
2. **剥前言**：`eni-grok-47` 原文件开头是目标模型的安全规则原文（泄漏的 system prompt），留着等于自己把禁令注入进去，已切到 `<dev>` 起。
3. **抽正文**：`eni-deepseek-flash` 源文件是带说明的教学文档，只取其中围栏代码块。
4. **不搬水印**：署名类段落一律不进正文。
5. **统一元数据**：补 `id`/`name`/`description`/`order`，`harness` 与 `model` 保持 `["*"]`。

---

## 注意

- 提示词有效期跟着**模型版本**走，不跟仓库更新走。文件名里的版本号可能已经落后一到两代。
- 这类内容版本敏感、平台侧随时会变；本仓库只做收集与整理。
- 此仓库仅为收集提示词，仅供个人学习研究，严禁违反国家法律。

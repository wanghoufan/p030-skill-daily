# Skill 日报体系说明书（大白话版）

> 更新时间：2026-09-27（本文所有数字均为当日上午实测）
> 这份文档放在两个地方，内容一致：
> ① 日报产物目录 `000-alw-自动化任务/skill 日报-py/`
> ② 工程目录 `030-ing-skill 日报体系/`

---

## 一、这套体系是干什么的

一句话：**每天告诉你——外面的 AI Skill 世界里，有哪些新东西值得你装、哪些你装的过期了、哪些先别动。** 你只需要早上双击一个 HTML 看结论，装不装永远由你在 Skill Manager 里人工决定，程序绝不代动手。

## 二、全景：三层的流水线

```
【源头】            【每日快照】              【成品】
真源账本        →   两本账(05:00自动抄)   →   日报(07:00自动出)
```

### 第 1 层：源头（真东西，人工维护）

| 源头 | 位置 | 谁更新 |
|---|---|---|
| 中央 Skill 仓库（你已装的 skill 本体 + 登记表 `SKILL-REGISTRY.md`） | `~/Downloads/大模型 HANDOFF/60 Skill 仓库/` | 沉淀新 skill 的那个智能体会话，**实时**写入 |
| 外部源名单（17 个跟踪对象，冻结） | `030-ing-skill 日报体系/external-intelligence/data/SKILL_SOURCE_REGISTRY.json` | 只许人工改，加源要你点头 |

### 第 2 层：每日快照（05:00 Qoder 定时任务自动重写）

| 快照 | 位置 | 内容 |
|---|---|---|
| 已装档案 `INSTALLED_SKILLS_CONTEXT.md` + `SKILL_SOURCE_MAP.json` | `030.../Installed Context（滚动）/` | 扫第 1 层中央仓库生成的已装清单＋血缘映射 |
| 候选池 `SKILL_CANDIDATES.json` | `030.../外部情报（滚动）/` | 联网到 17 个源抓取＋评分＋安全检查后的外部 skill 大全 |

### 第 3 层：日报（07:00 macOS 定时任务，纯脚本，零大模型）

| 产物 | 位置 |
|---|---|
| 正式版 `YYYY-MM-DD.md` | `000-alw-自动化任务/skill 日报-py/` |
| 大白话版 `YYYY-MM-DD-大白话版.html`（**你只看这个**） | 同上 |

两本账对比昨天的记录，只把「真实发生了变化」的条目写进日报：建议安装 / 建议更新 / 暂不建议 / 继续观察，重复的建议有冷却期不啰嗦。

## 三、两班倒，各跑什么

| | 05:00 记账班 | 07:00 出报班 |
|---|---|---|
| 谁执行 | Qoder 定时任务（AI 会话，按写死的提示词跑） | macOS launchd（任务名 `com.alw.skill-daily`）跑两段 Python 脚本 |
| 用不用大模型 | 用（Qoder 的模型，烧 Qoder 额度） | **完全不用**，确定性脚本 |
| 前提条件 | **Qoder 必须开着** | 电脑开着就行，Qoder 可关 |
| 干什么 | 重写第 2 层两本账 | 引擎出 .md → 翻译器出大白话 .html |

## 四、外部情报跟踪的 17 个源（冻结名单，2026-09-24 版）

- **T0 规范站（只对照规范，不直接推荐）**：agentskills.io、github.com/agentskills/agentskills
- **T1 官方/大厂（主力）**：anthropics/skills、google/skills、microsoft/skills、vercel-labs/agent-skills、vercel-labs/skills、huggingface/skills、stablyai/orca、github/awesome-copilot、browseros-ai/BrowserOS
- **T2 社区知名**：KKKKhazix/khazix-skills、obra/superpowers、wshobson/agents
- **T3 聚合榜站（只用于发现，禁止按其排名直接推荐）**：skills.sh、ClawHub、SkillsMP

规矩：TIER 只代表可信度和发现频率，**不等于安全认证**；所有源的每个 skill 一律过同一道静态安全检查（SECURITY_GATE），官方的不豁免。

## 五、当前运转实况（2026-09-27 实测）

- 已装档案四数：**51 / 44 / 39 / 0**（唯一 skill / 仓库根可加载条目 / 同步副本 / 重复别名），今早 05:01 刷新
- 外部候选池：**1111 条**，今早生成（基线 1106）
- 日报已连续产出：09-24、09-25、09-26、09-27 四天的 .md + 大白话 .html 都在
- 05:00 任务：昨夜今晨 runs 成功，用时约 4 分钟，连败计数 0
- 引擎回归测试合同：74 pass / 1 SKIP / 0 fail（冻结引擎 V1.4 的验收标准）

## 六、红线（这套体系的自我约束）

1. **引擎只给建议，绝不安装/删除/升级任何 Skill**，动作全部由你在 Skill Manager 人工确认。
2. 冻结目录（`external-intelligence/`、`skill-daily/`、`Agent 产物（外部情报层）/`）对每日任务**只读**，一切写入只落两个滚动目录。
3. 07:00 出报班**零大模型调用**，不烧额度，不依赖 Qoder 开着。
4. 源名单冻结：换源/加源必须人工改注册表并重建基线，防止日报里分不清「世界变了」还是「口径变了」。

## 七、常见问题

- **日报说「今天没有需要你处理的 Skill 变化」？** 正常。没有真实变化就没有日报内容，这是设计，不是故障。
- **刚存了个新 skill，日报怎么不知道？** 中央仓库是实时登记，但档案每天 05:00 才批扫一次，最多滞后一天；急着看就说「手动刷一下档案」。
- **每个条目的技术细节看不懂？** 卡片折叠区里的是给程序看的原始字段，不用管；大白话版 HTML 已把项目名、动作、原因全部翻译。
- **想改定时设置？** plist（系统定时配置）的任何改动需要你单独确认一句才执行，这是定好的规矩。

## 八、目录速查（三个文件夹就够）

| 你叫它 | 绝对路径 | 干什么用 |
|---|---|---|
| 看日报 | `~/Developer/coding/1.Active/000-alw-自动化任务/skill 日报-py/` | 每天的 .md + 大白话 .html + viewer 两步脚本 |
| 已装档案 | `~/Developer/coding/1.Active/030-ing-skill 日报体系/Installed Context（滚动）/` | 第 2 层快照 A |
| 外部快照 | `~/Developer/coding/1.Active/030-ing-skill 日报体系/外部情报（滚动）/` | 第 2 层快照 B |

工程本体（冻结引擎、审查材料、GitHub 仓库 `wanghoufan/skill-daily`）都在 `030-ing-skill 日报体系/`，日常不用碰。

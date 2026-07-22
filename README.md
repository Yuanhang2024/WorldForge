# WorldForge
**One Word + LLM + Framework = A Properly Progressing World**
 **一个 LLM 立法、确定性内核司法的 AI 世界模拟引擎。**
>
> 大多数 AI 世界生成器让 LLM 编故事——随机且不可复现。WorldForge 让 LLM 只起草规则，确定性内核执行规则。世界有物理必然性：三体文明逃离是被数学证明的，不是编的。

---

## 定位：立法与司法分离

现有的 AI 世界生成器把 LLM 同时当成**立法者**（定规则）和**法官**（判结果）。后果是世界失去必然性——退化成一台随机故事生成器：同一个 seed 跑两遍结果不同，用户"自信断言"就能改写真相，凶手是谁取决于上一句问了谁。

WorldForge 在编译期就切开这条边界：

| 角色 | 谁来做 | 管什么 |
|---|---|---|
| **立法（起草人）** | LLM（标准 OpenAI 协议，任意供应商） | 起草世界参数、规则草案、剧情叙述——只提议，不裁决 |
| **司法（法官）** | 确定性内核 | 物理积分、守恒校验、前提可达性、存亡判定——地面真相 |

这条边界就是全局红线 **I2**：**LLM 不得裁定任何有确定性真相源的事。** 一旦让 LLM 决定"这次预测成功了没有"，逃离就失去必然性。内核算出数字，LLM 只把数字翻译成剧情。

---

## 核心卖点

### 1. I2 红线：LLM 不当法官

状态变更绝不从 LLM 自由文本解析（**I3**），必须走结构化 tool call + 确定性校验。写入点唯一（**I4**）：90% 走规则引擎（<1ms 确定性），仅 10% 语义冲突才异步交给 LLM 裁定，且裁定必须写回状态。这不是运行时的自律，是编译边界强制的架构。


### 2. 以《三体》中三体文明的发展历程为例：LLM编写物理内核，逃离是数学证明的

- **辛积分器（Yoshida 4 阶）**——不用 RK4。百万模拟年尺度上 RK4 的能量漂移会制造"假混沌"，无法区分真物理与数值误差。
- **最大 Lyapunov 指数 λ** 由内核估计，不是设定值。
- **生存耦合闭环**：`科技 → 观测精度 δ₀ → 预测视界 T_pred = (1/λ)·ln(Δ/δ₀)`，视界能否覆盖下次乱纪元决定文明幸存 / 倒退 / 重启。
- **逃离证明**：当**最高科技下的最优预测视界**仍短于乱纪元平均间隔，理性上证明长期不可幸存——只能逃离。

```
$ python archive/scripts/evacuation_demo.py

seed=  7  λ=0.2128  乱纪元数=2  平均间隔=98.04  最优视界 T_pred=62.93  ==> evac_check=True

逃离被数学证明触发 (evac_check=True)：seed=7
  最优预测视界 T_pred_best = 62.93 < 乱纪元平均间隔 mean_gap = 98.04
  → 即便最高科技 (tech=10.0)，视界仍无法覆盖复发的乱纪元 (Lyapunov 墙 λ=0.2128)，
    文明证明长期不可幸存，只能逃离。
```

`evac=True` 不是 LLM 写的一句话，是 `T_pred_best < mean_chaos_gap` 这个不等式在 seed=7 的时间线上成立。

### 3. 世界隔离

`world_id` 命名空间，多世界同时存在互不串扰。每世界独立状态文件、独立烘焙时间线、独立仪表盘。构成性约束（λ、物理、初值）创建后不可变——要变只能 **fork 新世界**，不能 mutate 正典。

### 4. 全确定性内核测试覆盖

确定性内核全部可单测（秒级、必须全绿）：物理能量守恒、DAG 无环、事务快照/提交/回滚可复现、规则引擎 5 项校验、96 条黄金脚本回归。内核测试不过 → 不允许合并。

### 5. 全本地全开源 + 任意 LLM 供应商

单机运行，代码全开源。LLM 通过标准 OpenAI `/v1/chat/completions` 协议连接——可以是本地 LM Studio/Ollama/vLLM，也可以是云端 OpenAI/Anthropic/任意代理。仅需配置 `base_url` + `api_key`。CLI 路径完全确定性、可离线复现。

### 6. 确定性涌现：条件触发 + 事件链

`WorldKernelHost` 支持声明式条件触发和事件链：规则不是每 tick 无条件执行，而是当状态越过阈值时自动点火。infected > 50 → 自动隔离 → demand 骤降一半；demand < 30 → 自动刺激 → 医疗资源增加。**没有人干预，世界自己产生连锁反应**——同 seed 每次运行一模一样。一次性门控保证链不循环，守恒规则保证涌现不造钱。

```python
auto_quarantine = WorldRule(
    rule_id="auto_quarantine",
    kind="dynamic",
    trigger={"op": "and", "args": [
        {"op": "gt", "args": [{"ref": "infected"}, {"const": 50}]},
        {"op": "not", "args": [{"ref": "policy.quarantine"}]},
    ]},
    effect={"emit": "quarantine_started", "writes": "policy.quarantine",
            "op": "set", "value": True},
)
```

### 7. 多内核组合：物理+经济同 tick 运行

`MultiKernelHost` 是一个微内核骨架 + 事件总线 + 波次调度器，支持在同一 tick 内按优先级依次执行多个内核。事件通过总线跨内核传递：物理内核的 `civilization.destroyed`（乱纪元）触发经济内核需求下降 20%——灾难有物理必然性，经济影响有确定性因果关系。

- **kernel write own**：装配期校验 owns_paths 无重叠，运行时禁止越界写
- **事件总线**：pub/sub，最大 5 波级联防无限循环，同 tick 内原子传递
- **波次调度**：priority 排序，先物理后经济
- **确定性**：同 seed 两次独立运行轨迹完全一致

```python
host = MultiKernelHost([physics_module, economy_module], state, seed=42)
host.tick(external_events=[time.accelerate, turn.tick])
```

### 8. 运行时世界内核：规则 DSL → 编译器 → 确定性执行

`WorldKernelHost` 让 LLM 起草 JSON DSL 规则（如生产、消费、价格调整），`RuleCompiler` 编译为确定性 Python 内核并执行——LLM 绝不触达结果（立法/司法分离）。内置 AST 操作白名单（四则运算、比较、逻辑、lookup），杜绝注入。

经济世界 demo 证明：WorldKernelHost 能跑**任意世界**，不限于我们预装的 ThreeBodyKernel 和 DeckKernel。


---

## 架构

```
用户层
   ↓
交互显示层     ST-前台 (角色卡/多模态输入) + Web 舞台 (HD-2D 渲染/动画/音频总线)
   ↓
网关与准入层   FastAPI 网关 + 带宽准入信物 + 模型调度器
   ↓
智能层
 ├─ 实时路径（无 LLM）    意图 kNN <10ms · 规则引擎 <1ms · 可供性查表 <1ms
 └─ 推理路径（骨干 LLM）  叙事规划 / 语义裁定 / 换皮（batch 服务 + 前缀缓存）
   ↓
═══════════ 立法/司法边界（I2）═══════════
   ↓
世界内核层（确定性，可单测）
   三体物理内核 · 前提 DAG · 状态事务 · 规则引擎 · 玩法内核库
   ↓
数据层         世界书 · 角色实体库 · 内容寻址资产库
   ↓
持久层         存档(schema 版本) · 迁移链 · 完整性标记
```

四台通用机器覆盖全部场景类（不写死在三体项目里）：

| 机器 | 覆盖 |
|---|---|
| 内核库 + 检索 + 换皮 | 玩法、动画、资产、场景模板 |
| 前提 DAG + 可达性校验 | 认知进展、配方/建造、任务前置 |
| 状态事务（快照/提交/回滚） | 时间加速、反事实、投机缓冲 |
| 守恒/不变量校验 | 经济、资源、库存、人口 |

---

## 模块清单

**世界内核（确定性核心）**

| 模块 | 职责 |
|---|---|
| `kernels/three_body.py` | 辛积分 + Lyapunov + 生存闭环 + 逃离证明 |
| `dag.py` | 前提 DAG（认知链/配方树通用），可达性查询 |
| `transactions.py` | 状态事务：快照 / 提交 / 回滚 |
| `rule_engine.py` | 规则引擎：5 项确定性校验 + LLM 裁定拆分 |
| `game_kernels.py` | 参数化玩法内核库（结算零 LLM） |
| `gates.py` | 校验门 |
| `multi_kernel.py` | 多内核组合：微内核骨架 + EventBus + MultiKernelHost + 波次调度 |
| `world_kernel_host.py` | 运行时世界内核主机：规则 DSL → RuleCompiler → 确定性执行 + ExprEvaluator |

**世界生命周期**

| 模块 | 职责 |
|---|---|
| `world_creation.py` | 世界 propose / preview / create / fork |
| `world_import.py` | ST 卡片导入编译 + 注入防护 |
| `migrations.py` | schema 版本 + 迁移链 |
| `storage.py` | JSON 状态存储（world_id 隔离） |
| `idle_loop.py` | 空闲循环（世界自主推进，带饱和上限） |
| `save_manager.py` | 存档管理：save / load / list / verify / delete（zip + schema 迁移） |
| `gui_generator.py` | GUI 生成：每世界自定义 UI（propose+confirm 流程，LLM 提议→用户编辑→确认） |
| `onboarding_*.py` | 9 个 propose+confirm 端点：每步 LLM 预填表单 → 用户审查编辑 → 确认。覆盖规则/GUI/认知模型/玩法 |

**智能层与推理优化**

| 模块 | 职责 |
|---|---|
| `dispatcher.py` | 编排器：wire 内核 + LLM + 资产 |
| `agents.py` | 模型客户端（含 `DeterministicModelClient` 离线态） |
| `llm_optimizer.py` | 推理优化总入口 |
| `prefix_cache.py` | RadixAttention 前缀缓存 |
| `best_of_n.py` | Best-of-N 采样 + 重排 |
| `speculative_decode.py` | 投机解码（draft + target） |
| `lookahead_buffer.py` | 前瞻缓冲 + 质量预算降级 |
| `capability_report.py` / `provider_diagnostics` | 能力探测 / 供应商诊断 |

**资产与准入**

| 模块 | 职责 |
|---|---|
| `content_addressed_assets.py` / `assets.py` | 内容寻址资产库（哈希去重 + 溯源 + 版本钉死） |
| `asset_prefetch.py` | 叙事前瞻驱动的资产预取 |
| `admission.py` | 带宽准入信物（全局唯一重型消费者信物） |
| `local_media.py` / `voice_bank.py` | 本地媒体 / 音色身份库 |

**方块人视觉（`chardoll*` + `rehearsal_compiler` + `animation_library` + `web/` + `fork/`）**

11 骨方块人角色 · 表情纹理 · 骨架自适应 · 排演编译器（乐谱 → 动画时间线） · HD-2D 舞台（光照 + 景深） · 像素画 fork。

**服务与配置**：`server.py`（FastAPI）· `runtime.py` · `settings.py` · `models.py`

---

## CLI 用法

```bash
worldforge "三体世界"                    # 创建世界 + 预览（内核预跑推演）
worldforge "三体世界" --tick --tech 5.0  # 推进一拍（带科技决策）
worldforge "三体世界" --tick             # 推进一拍（延续上次决策）
worldforge "三体世界" --bake             # 烘焙 + 输出仪表盘数据（λ/纪元/逃离/光照）
worldforge --list                        # 列出所有世界
worldforge "三体世界" --save [out.zip]   # 导出存档
worldforge --load save.zip              # 导入存档
worldforge --saves                      # 列出所有存档
worldforge --demo                       # 跑完整演示流程（三体世界→推进→逃离证明→存档）
python archive/scripts/evacuation_demo.py        # 逃离证明 demo
python archive/scripts/economy_demo.py           # 经济世界 demo（运行时 DSL 内核）
python archive/scripts/card_game_demo.py         # 牌局世界 demo（DeckKernel 第二世界）
python archive/scripts/multi_kernel_demo.py      # 多内核组合 demo（物理+经济同 tick）
```

CLI 是纯 Python（argparse）入口，接离线 `DeterministicModelClient` 并 stub LLM 叙述——**每条命令本地确定性运行，不对外部 LLM 服务有硬依赖**。逐拍 tick 状态持久化进状态文件 `_meta`，连续 `--tick` 跨进程真正推进时间线。

**世界预览是本架构独有、LLM-only 系统抄不走的杀手锏**：提交前用确定性内核把世界跑一遍，把 `λ=0.003` 翻译成"文明平均存续 N 年·你会见证 3-5 次兴衰·最早第 4 号文明逃离"，让最大的工程投入变成最大的 UX 资产。

---

## 测试

```bash
python -m pytest tests/          # 确定性内核 + 全后端回归
python -m pytest fork/tests/     # 方块人视觉 fork
```

- **确定性内核套件**：976 项（`tests/`），全绿是合并的阻塞门
- **黄金脚本回归**：96 项固定输入序列（含刁钻用例），跑完检查不变量未被破坏
- **视觉 fork**：68 项（`fork/tests/`）
- 覆盖：辛积分保能量 · DAG 无环 · 事务可复现 · 规则引擎 · 世界隔离 · 世界创建/导入 · schema 迁移 · 推理优化（前缀缓存/best-of-N/投机解码）· 骨架/动画/口型对齐

> 内核部分完全确定性，测试成本极低收益极高；LLM 部分只能抽样评估。

---

## 设计文档索引

| 文档 | 内容 |
|---|---|
| [`docs/ARCHITECTURE_SPEC(2).md`](docs/ARCHITECTURE_SPEC(2).md) | 主架构规范：7 条红线 · 硬约束预算 · 分层架构 · 三体内核 · 世界内核 · 权限模型 · 安全边界 · 构建顺序 · 公共数据结构 · 禁止清单 |
| [`docs/runtime_world_kernel_design.md`](docs/runtime_world_kernel_design.md) | 运行时世界内核设计 |
| [`docs/gui_generation_design.md`](docs/gui_generation_design.md) | GUI 生成设计 |
| [`docs/visual_pipeline_audit.md`](docs/visual_pipeline_audit.md) | 视觉管线审计 |
| [`docs/MODEL_ROUTING.md`](docs/MODEL_ROUTING.md) | 多模型路由：per-role 默认模型 + 环境变量覆盖 + 标准 OpenAI 协议供应商配置 |

---

## Demo 清单

| 脚本 | 内容 |
|---|---|
| `archive/scripts/evacuation_demo.py` | 逃离证明 demo：扫 seed 找 `T_pred_best < mean_chaos_gap` 的数学证明触发点 |
| `archive/scripts/emergence_demo.py` | 涌现 demo：条件触发 + 事件链，infected > 50 → 自动隔离 → demand 骤降 → demand < 30 → 自动刺激 → 医疗增加，无人干预 |
| `archive/scripts/economy_demo.py` | 经济世界 demo：运行时 DSL 编译内核，10 回合供需 + 价格 + 守恒 + DAG 制度解锁 |
| `archive/scripts/card_game_demo.py` | 牌局世界 demo：DeckKernel 第二世界，4 玩家 13 墩全手牌，确定性叙事 |
| `archive/scripts/multi_kernel_demo.py` | 多内核组合 demo：物理 + 经济同 tick 运行，事件跨内核传递，owns_paths 隔离 |
| `worldforge.py --demo` | CLI 统一演示：创建世界 → 预览 → 3 拍推进 → 逃离证明 → 存档 → 列存档 |

---

## 为什么这不是又一个 AI 故事机

`sudo set lambda 0.001` 之后，逃离不再是被证明的，它变成打字打出来的——更像"要求获得国际象棋规则的 sudo"：车能斜走了，但你已经不在下棋。

WorldForge 对用户的承诺是：**你不能改写这个宇宙，但你可以创造任何宇宙。** 你拿不到 mutate，但你拿得到 λ 的整条谱系——看不同混沌强度下文明命运的分布。自由度实际更大。

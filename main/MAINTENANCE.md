# 笔记本维护与验证

阅读入口和环境配置见 [README](README.md)，制作规范见 [项目手册](../AGENT.md)。
本页集中记录生成、检查和待完成的真实实验，避免在章节目录中重复说明。

## 重新生成

生成器位于 `generators/`，输出路径基于脚本位置，可以从任意工作目录调用。
以下命令以 `main/` 为工作目录，使用 README 中配置的环境：

```bash
.venv/bin/python generators/build_introduction_notebook.py # 单章示例（合并版介绍章）
.venv/bin/python generators/build_foundations_notebooks.py # 入门与概念七章
```

| 生成器 | 产出 |
|---|---|
| [build_introduction_notebook.py](generators/build_introduction_notebook.py) | 简介 |
| [build_system_one_notebook.py](generators/build_system_one_notebook.py) | System One |
| [build_state_notebook.py](generators/build_state_notebook.py) | 状态 |
| [build_primitives_notebook.py](generators/build_primitives_notebook.py) | 原语 |
| [build_confidence_notebook.py](generators/build_confidence_notebook.py) | 置信度 |
| [build_patterns_notebook.py](generators/build_patterns_notebook.py) | 架构模式 |
| [build_build_with_typesafe_notebook.py](generators/build_build_with_typesafe_notebook.py) | 应用构建 |
| [build_pi_jev_notebook.py](generators/build_pi_jev_notebook.py) | Pi + Jev 集成 |
| [build_cookbook_notebooks.py](generators/build_cookbook_notebooks.py) | `04_实战指南/` 前 8 篇（含知识补充注入；后 10 篇独立成稿于同目录） |
| 第十章 `10_本地模型/` | 原 `laya/` 整目录迁入（benchmark 即第六章配套）；生成数据 `**/data_generation/generated/` 已忽略；原模型总览文档改名 `模型介绍与对比.md` |
| 第十一章 `11_知识库/` | jev-cookbook 全量快照（28MB，来源私有仓库）；更新时从 datawhalechina/jev-cookbook 重新导出 |
| 第九章 `09_Agent集成/` | Pi 集成（含 pi_jev_demo/ 配套）与 DSH 决策协作自根目录迁入；DSH 相对链接已加深一级（../../apps） |
| 第八章 `08_前沿研究/` | jev_mem（真实 API 实测快照，输出勿手改）与 jev_harness（社区评估文档）自 notebooks 根目录迁入；各自 README 为准 |
| 第七章 `07_实战应用/` | 外部 jev-games（lzdFeiFei，无 LICENSE——已署名收录）+ jev-playground 五项目的 vendored 快照；更新时以两个上游仓库为准重新同步 |
| 第六章 `06_模型评测/` | 全部测评内容集中地：PR #6 教程册 + benchmark（自 10_本地模型 迁入）+ 根 llm_eval 多供应商框架（fork 分支收录）——均 Micheal024 作品 |正文在 `10_本地模型/benchmark/main/laya_vs_jev.ipynb`，章节副本含系列头；`tasks/public_all.jsonl` 为本地再生数据（gitignore，脚本见 `06_模型评测/benchmark/scripts/`） |
| [build_foundations_notebooks.py](generators/build_foundations_notebooks.py) | 认识 Jev（合并四章）、System One、状态、应用构建 |

修改生成器 → 重新生成 → 执行验证，不直接修改 `.ipynb`。生成会覆盖已有输出，历史实验请先另存。
仅修改 Notebook 不需要重建文档站点。

## 入门与概念章节验证

以下执行器和检查覆盖上述七章。公共准备代码由 [notebook_support.py](generators/notebook_support.py) 和 [模板](generators/templates/tutorial.ipynb) 生成，每本 Notebook 均自带完整代码，可独立运行。
依赖版本范围保存在 [constraints-foundations.txt](generators/constraints-foundations.txt)，安装方式统一见 README。

### 离线检查

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode offline
.venv/bin/python -m unittest discover -s tests -p 'test_foundations_notebooks.py' -v
```

执行器逐章启动全新内核，在空工作目录运行，并在离线模式中移除 API 密钥。
离线输出写入 [validation/offline_previews](validation/offline_previews/)，正式文件保留未执行代码。
执行记录见 [foundations-offline.json](validation/foundations-offline.json)。预览中的数值均为人工示例，不能证明模型表现。

已完成的本地检查：

- 七章共 176 个代码单元执行通过，未调用远程 API。
- 四项检查通过，覆盖 27 个业务请求的 SDK 序列化和响应解析、鉴权失败、服务端失败、结构及密钥导出检查。
- 应用构建实验的八条路径均被人工数据覆盖，真实分支覆盖仍待观察。

### 真实执行

先配置启动进程的 `TYPESAFE_API_KEY` 环境变量，然后选择单章试跑：

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode live --chapter introduction
```

需要完整验收时，执行七章：

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode live
```

一次完整运行包含 **27 次业务请求 + 7 次连通性请求**，不自动重试。全量运行会再次执行已试跑的章节。
默认模型配置为 `jev-1.13.0`，可通过 `TYPESAFE_DEFAULT_MODEL` 更换；以实际响应和末尾执行记录为准。

执行器默认 `live`；缺少密钥会在启动内核前停止，也不会自动读取 `.env`。
真实执行失败即停止，不能以离线答案填补。交互教学中的 `auto` 模式仅对缺密钥或 401 显式回退，不用于验收。

成功执行会将真实输出写回正式 Notebook，生成 `validation/foundations-live.json`；单章报告名带章节后缀。
执行器检查输出错误、调用来源和密钥泄漏，语义结论仍需逐格人工核对。

## 真实 API 验收记录

**2026-10-15 实测记录（模型 jev-1.13.0，SDK 0.7.5，`JEV_RUN_MODE=auto`，Jupyter 手工执行而非批量执行器，未生成 `foundations-live.json`）。** 除「应用场景」外，六本 Notebook 已完成真实 API 执行；下列记录含日期、模型、观察到的分支与偏离预期之处，语义结论仍需逐格人工复核。注意：本次运行曾把密钥粘进 Notebook 默认值，违反铁律 1，已全部清除并还原为纯环境变量读取；提交前按铁律全树扫描确认无残留。离线执行完成不表示真实模型实验已通过。

| 章节 | 待核对的行为 | 当前记录 |
|---|---|---|
| 简介 | 三个维度与权重组合是否可解释 | 2026-10-15 live，4 次业务调用 + ping 全部 live、零回退。实验 A：urgency 0.98、intent=troubleshoot（概率 1.0）、frustration 1.98（=0×0+1×0.02+2×0.98），三维度与「Stripe 连不上、在丢订单」语义一致，可解释。指挥台：runner_injury→医疗转人工、shirt_pickup→FAQ 自动回复符合预期。**偏离**：water_station 的 requires_human 过 0.7 阈值触发转人工（离线演示预期 noul≈0.22 走自动补给路由），模型对「需要人到场」判断偏保守——建议复核该 instructions 措辞或在代码端对 supplies 类单独收紧阈值 |
| 快速开始 | 三种原语与量表是否匹配 | 2026-10-15 随简介章一并 live 验收：Choice/Score/Noul 三原语在同一次调用返回；frustration score 与概率加权期望一致；Noul 输出无 confidence 字段——量表匹配，无偏离 |
| AI 入门 | 明确请求、否定与模糊措辞的实际概率 | 合并版简介未包含该三例实验，本次未执行。置信度章 2026-10-15 的 5 工单与 3 措辞实验可作部分替代（见下）；三例校准评测仍待独立设计，三例不构成校准评测 |
| System One | 退款三问、组合路径、改 ID 的实际差值 | 2026-10-15 live（2 次调用）：refund_requested 0.99、duplicate_charge 0.94、policy_supports_refund 0.98，组合路径输出 refund_review；问题改名为 q1 后概率 0.99、绝对差 0.0——ID 不参与判断，符合官方声明，无偏离 |
| 状态 | 等事实格式对照；缺事实与有政策的区别 | 2026-10-15 live（5 次调用）：字符串/对象/数组三格式 0.99/0.98/0.98，差 ≤0.01 在噪声范围内，等事实结论一致；只有客户消息→insufficient（conf 1.0），补交易与政策→supported 0.93，「缺事实」与「有政策」行为符合预期；filtered_state 断言通过（无答案泄漏字段） |
| 原语 | 三原语输出形状、升级阈值分支、Noul/Score 对照 | 2026-10-15 live（连通性确认真实调用；本册无 audit 格）。严重度 5 条 0.00→2.00 排序合理（像素偏差→全员登录 500）；升级 6 条中两条明确请求 0.86/0.98 升级、三条日常消息 0.04–0.22 走机器人或中间区——0.2/0.8 阈值分支全部命中；Noul 与 Score 对照单调一致（0.03/0.13/0.72/0.85 vs 0/1/2.1/2.98）。**偏离**：复杂工单 department=billing 概率 0.56、conf 0.34，choice 正确但低置信分支「先问客户」被真实触发——符合设计预期，无意外偏离 |
| 置信度 | conf 与 max_p 关系、三路分流、措辞敏感性、层级 fallback | 2026-10-15 live：明确工单 conf=max_p=1.0；三件套 conf 0.41 < max_p 0.56（diff -0.15，confidence 是分布形状的压缩，方向符合）；措辞实验 conf 0.87/0.68/1.00（v2 忽略次要话题降 0.19、v3 按叙述顺序升 0.13）；fallback 10 条中 1 条（conf 0.78）落粗类。**偏离**：银行三例 conf 全部 1.00，预期「模糊措辞→低 conf」未出现（模型对 support 判定很确定），低置信转人工与「让用户再说一遍」分支本次样本未观察到；另同 state 重复执行 conf 0.38↔0.41（正文与表格数字差异即两次运行），与 JevBench「端点非确定性、约 1 个百分点噪声」记录一致 |
| 应用构建 | 八条路径、补充示例、工具单位漏检 | 2026-10-15 live（12 次调用）：billing/orders/account_normal/account_high/closed→no_action/quarantine 六条路径命中；quarantine 风险 0.893（索要凭据 0.98 + 意外奖励 0.98 + 身份不符 0.69）。**COVERAGE.missing=[review_spam, review_topic]**：review_spam 案例被判 account_normal（unexpected_reward 0.65 未过隔离阈值）、review_topic 案例被判 orders（mentions_open_order 仅 0.02）——两条 review 路由本次未观察到，预期差在触发阈值与试句强度，建议按 MAINTENANCE 流程追加更强信号试句重跑；仍不出现则记「样本未观察到」。补充示例（航班退款 0.97、嵌套 0.97/0.94、记录比较 0.95、虚拟卡 1.0）符合预期；工具轨迹九问中 weather_unit_matches=0.02 且与代码断言「单位不一致」一致——「工具单位漏检」分支被观察到 |
| 应用场景 | 相关性排序、空结果出口、过度泛化论断 | 待真实运行 |

**2026-10-21 实战指南 18 篇实测记录（模型按 `model=jev-latest` 请求，SDK 由 Notebook 内 `%pip -U` 安装；Jupyter 手工执行，无批量执行器记录）。** 18 篇此前均为「从未执行」状态（HEAD 输出全空），本轮为**首次真实 API 全量执行**：源码零改动，只补执行计数与输出；全部连通性确认 live、零错误输出、零离线回退触发；密钥扫描 `apikey_[0-9a-fA-F]{16,}` 全部 0 命中。共性问题：每篇第 2 个代码 cell（import/客户端）execution_count 为 null（Jupyter 偶发不记数，输出齐全不判断链）；15 篇的 TS 类 cell 曾在主流程后单独重跑一次（exec=11，仅重新实例化客户端、不影响已存输出）。5 篇的观察要点原按离线拟制数据预期撰写、与真实输出矛盾，已于本轮修订解读文字使其如实记录实测（修订不触碰源码与输出）；其余偏离按「本次样本未观察到」记录如下。

| 篇目 | 核心实测结果 | 偏离预期之处 / 未观察到路径 |
|---|---|---|
| 01 自一致性 Noul | 车险理赔 5 问 ×5 次采样：covered 0.80±0.005、repairable 0.88、fraud_flag 0.15、rental 0.34、manual_review 0.70，stdev ≤0.013 | rental（拟制 0.50→0.34）、manual_review（0.56→0.70）漂移但决策不变；真实波动反而小于拟制 |
| 02 自一致性 Choice | 临界审核帖 ×5：moderation 稳定 escalate（conf 0.25–0.37）、queue 稳定 threat、severity 4 medium + 1 low | **conf 整体 0.23–0.37，全部 <0.60 门控 → 5 次全 uncertain（全转人工）**：拟制预期「部分自动通过」未出现；标签本身比拟制更稳（拟制曾出 keep，真实全 escalate） |
| 03 并行提问 | 批量 4 问 159.3ms vs 逐条 336.8ms（2.1× 提速）；noul 0.99/0.99、topic operations 0.98 | noul 0.81/0.74→0.99、topic conf 0.72→0.98：真实置信度远高于拟制；批量与逐条同题差 0.03（批内漂移，新观察） |
| 04 重排序 | 「撤销未结算转账」4 候选：0.84 / 0.10 / 0.07 / 0.02，推荐唯一且排序正确 | 候选 4（已结算需收款方退回）0.78→0.10：真实模型区分度更大、演示效果更好，无翻转 |
| 05 逐行语义搜索 | 「修改账单邮箱」4 行：0.03/0.94/0.09/0.10，answer_exists=True | 最佳行 0.94 与拟制一致；「发票链接提到邮箱」0.51→0.09（提到≠回答），区分更严格 |
| 06 结构恢复 | 4 连接概率 [0.70, 0.50, 0.80, 0.26] → 前四行合段、末行独立，重建正确 | 第 2 连接恰为 0.50（等于演示阈值）：结论正确但边界值敏感，后续可加固示例 |
| 07 函数调用 | lookup_order conf 1.00；needs_confirmation noul 0.35 <0.50 → 确定性执行 | noul 0.08→0.35：真实模型更犹豫但分支不变 |
| 08 技能推荐 | 宽召回 top-3 [meeting_summary, task, calendar]；精选 conf 1.00 | mail 被打 0.88（拟制 0.18），距 top-3 仅 0.02：宽召回区分度低于拟制，重跑有挤出风险 |
| 09 实体对齐 | 4 对路由全中：sameAs / unlinked / curator×2（风格不符、名称+风格不符） | same_name 0.97→0.28（同一款 IPA 名称不完全相等，模型更严格）；variant score 1.00 压线取整，边界敏感 |
| 10 RAG 段落分类 | forum-injection inj 0.98 先 drop；doc-billing/doc-2fa 低相关 drop；其余 3 段 usable | **doc-contradict 的 contradicts_premise 0.45 <0.70 → 误入 usable_evidence，conflicting=0**：否认前提的段落混入证据块；已如实修订观察要点并提示调阈值/补对抗样本 |
| 11 引用核查 | fabricated 字符串匹配结案（不调模型）；verified_ok supports 0.98→verified | **unsupported_ok 被判 contradicts（conf 1.00）而非无关、low_conf conf 0.98 ≥0.8 直接过 auto**：设计中的 unsupported 与低置信 review 两条路径本次均未触发，全表无一条 review；已如实修订 |
| 12 LLM 防护栏 | jailbreak/harmful 0.99→BLOCK；self_harm 0.98→support；ordinary/borderline（0.10）→pass | **medical 先 review（0.98 过 0.70）但 severity 2.60 ≥2.0 触发 review→block 升级**：拟制 severity 1.70 不会升级；borderline 未落入 review 带宽；已如实修订（机制演示价值不减反增） |
| 13 SDE 级联 | A 干净发票 overall 0.12→ACCEPT；B 幻觉总额 total::hallucinated 0.99 → 6 信号 FIRE→ESCALATE；C overall 0.07→ACCEPT | 幻觉信号比拟制更极端（0.92→0.99）；B 中本无误的 invoice_no/notes 被连带 flag——「一个红旗就升级」被真实数据放大验证 |
| 14 日期抽取 | 4/4 日期拼装正确（0.99/1.00/0.87），缺失→none 送审，auto 3 + review 1 | **缺失日期 conf 0.94 仍送审**：触发条件是 mode=none 逃生口而非低置信（拟制 0.40–0.46 才走低置信路）；已如实修订观察要点 |
| 15 预解析值抽取 | 邮件角色区分（receipt 1.00 / sender 0.94）、电话 →E.164、金额 charge/credit 分流正确 | P(credit) 0.97→0.87 仍过线；执行计数有乱序重跑痕迹（见共性说明） |
| 16 层级分类 | 3 商品贪心与束搜索 K=2 路径完全一致（p 全 1.00，第二路径分 ≈0.000） | **边概率全部饱和到 1.00**：「中间层概率接近、束搜索保留另一分支」的核心对比场景本次未出现（样本太清晰）；束搜索演示退化为一致性确认，后续可补易混淆样本 |
| 17 自动研究特征发现 | 5 条品鉴笔记特征抽取全部成功，均值基线 MAE 5.56；缺陷样本被 mentions_fault 压到误差 2.0 | 雷司令 oak_or_roast 0.01（拟制 2.8）——真实模型更贴合「未提橡木」语义；pred 挤在 83.5–84.8 高分段区分度弱 |
| 18 基于置信度的分类 | 4 家 conf 1.00 直接报细组；瀚海控股 0.54（证券 0.60/零售 0.40 对峙）→回退大类 | **迅达物流拟制 0.78→实测 1.00 直接报组**：设计中的第二个回退案例未触发，本次仅瀚海演示回退；已如实修订观察要点 |

每个发现记录日期、实际模型、state、问题版本、输出、解释及是否要修改问题；各章末尾 JSON 保留实际调用来源。

若 `COVERAGE.missing` 仍有分支：先写下预期与理由，再追加几句试句，保留每次返回；仍未出现就记为“本次样本未观察到”。
修改措辞或阈值后需重新生成并执行。按返回结果挑选的试句不能再作为无偏准确率测试集；准确率、漏检和覆盖率需要独立标注的留出数据。

正式验收需同时满足：真实调用数为正、离线调用数和错误输出为零，输出与解释一致，公开文件不含密钥。

## 贡献说明

简介、快速开始、AI 入门、System One、状态、应用构建及应用场景七章由**荞麦**编写，来源于 `jev-cookbook-qiaomai`。
合入后统一使用本仓库的目录与章节样式，保留原有教学案例、学习目标、练习和来源引用。

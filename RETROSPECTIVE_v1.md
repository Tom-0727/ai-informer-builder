# AI Informer v1 构建回顾

本文档是 AI Informer 第一版（从 2026-05-05 设计起步到 2026-05-07 cron 上线）的事后回顾。范围限定于人机协作矫正轨迹与一次系统化瘦身评估，不修改任何其他文件，不执行所列优化建议。证据来源是 mailbox/human.jsonl 与 Memory/episodes/2026/05/ 下 24 份执行记录。

## 第一节：人机协作矫正轨迹

下面按发生顺序列出 11 条 human 主动矫正的实例，每条给出 WHAT/WHY/HOW，再附一组次要修正。

### 矫正 A：把 Builder 运行机制误套在业务 Agent 上

WHAT：v1 设计稿把 heartbeat、mailbox、episode、todo_list 这些 Builder 的运行时概念写进了 AI Informer 自身的运行模型，screener 与 researcher 的调用关系也含糊不清，被 human 在 mail.20260505T102039Z.001 直接判定"完全错了"。证据：ep.20260505T090000Z 即 v1 设计 388 行成品。

WHY：缺乏对 Codex-based 业务 Agent 与 Builder 自身运行框架的边界认知；当时没有先打开 build-a-codex-agent 这个权威 SKILL.md 再动笔，而是把熟悉的 Builder 词汇直接迁移过来。属于"缺乏明确指引主动调用"加"缺乏对应分层模型的先验知识"叠加。

HOW：以 build-a-codex-agent 的三层心智模型为骨架重写——主代理在 AGENTS.md，技能在 .agents/skills/，子代理在 .codex/agents/<name>.toml 由主代理在单次 Codex run 内联调用，cron 是唯一调度驱动。后续把"声明完成前 grep 一遍 Builder harness 词汇"沉淀为 heuristic--codex-agent-design--audit-builder-harness-vocabulary.md。重写产物在 ep.20260505T102200Z 落地为 67 行版本。

### 矫正 B：硬性 3-5 条 / 凡是确认型消息必 await-reply

WHAT：mail.20260505T105222Z.001 给出六条具体修订，包括把每窗口 3-5 条改成 up to 15、把 §2 标题改名为 Architecture Design、recommendation-rendering 这一名字毫无意义、Q2 反馈回写完整删除、明确 Feishu 群 webhook 作为投递通道；同时强制要求"未来要等我确认的消息必须用 --await-reply"，并指示写进 CLAUDE.md。

WHY：第一版规则措辞过软（"如果需要暂停就用 --await-reply"），导致我把上一个需要确认的消息发成了 await_reply false。这不是模型能力不够，是 CLAUDE.md 规则颗粒度不够明确。

HOW：把 CLAUDE.md mailbox 段升级为强制句式"When sending a mailbox message that requires the human's confirmation or decision, you MUST use mailbox-send with --await-reply"。本次的回应 mail.20260505T105959Z.001 立刻按新规则使用 --await-reply，这条规则后续在三轮设计修订与多轮构建确认中被反复实践。详见 ep.20260505T110000Z。

### 矫正 C：删除可调旋钮类语言、删除 HN 反查放大器

WHAT：mail.20260505T132212Z.001 指出设计中保留的 rubric weights / threshold / max_shortlist_size / depth_budget / framing prefix（"今晨重点 / 中午观点 / 晚间技术"等）这一类参数化设计与 LLM 语义筛选的本质冲突，并要求删除 HN reverse-lookup amplifier，让 HN 回归为 08 窗口的普通来源。

WHY：底层错误是把基于 LLM 的 screener 当成"可调权重的传统推荐系统"来设计，没意识到 Agent 推荐的核心是清晰的语义指引而不是数值旋钮。属于建模认知错误。

HOW：在 ep.20260505T132500Z 把 §4.1 / §4.2 task packet 改写为"窗口主题 + 自然语言相关性判据 + recency 直觉 + prior_recommended_dedupe_keys"，HN 改为 12 个 08 窗口源中的一个，并新增 11 词禁用 grep 闸门（rubric weights / threshold / max_shortlist_size / depth_budget / framing prefix / 三个中文 framing 串 / reverse-lookup / amplifier）防止旋钮"以举例形式"溜回去。

### 矫正 D：subagent 接口必须是自然语言、screener 必须做文件 IO

WHAT：mail.20260505T142421Z.001 指出 subagent 设计仍在用 strict-JSON return 契约约束 LLM，违背 inline subagent 的本质；同时 screener 不应直接读完整 collected.json 的 JSON 结构，而应通过脚本读取剥离过的纯文本列表，再写出筛选后的 JSON 文件，并要求每次运行都有一个中间产物目录。

WHY：JSON 返回契约对 LLM 子代理没意义、只对 on-disk 数据有意义这一区别，前一版没看清；同时上下文预算约束没有被显式建模——把完整 collected.json 塞进 screener 的 context 是隐式失败模式。属于设计模式认知遗漏。

HOW：在 ep.20260505T143000Z 把 design/subagents/{screener,researcher}.md 改写为三段自然语言（input from main agent / internal behavior / return）；引入共享辅助脚本 .agents/skills/_shared/scripts/strip_for_screener.py 做剥离；锁定 Runtime/runs/<window>/<YYYYMMDD-HHMM>/ 作为每次运行的隔离目录，含 collected.json、stripped.txt、shortlist.json、briefings/<record_id>.md、output.md 五件套。

### 矫正 E：通知通道必须设计进系统而非临时 POST

WHAT：mail.20260505T153355Z.001 要求把飞书群通知作为可扩展的设计层正式融入，参考 https://github.com/Tom-0727/AInformer/blob/main/core/utils/inform.py 的实现，未来还要扩到其他通道，且 agent 调用应是简单 CLI、默认 broadcast。

WHY：v1 在 skill Stage 4 直接写了"POST 到 webhook"，把通知当成临时副作用而不是独立子系统。这是在缺乏明确指引时的简化偷工。

HOW：在 ep.20260505T154500Z 新增 design/notifications.md，落地 ai_informer/notifications/ Python 包：cli.py 入口 + registry.py 注册式发现 + channels/<type>.py 适配器，--message-file 通过文件路径而非命令行参数传入避免多行中文引号问题。webhook URL 唯一归宿改成 config/notifications.json 配置文件中的 webhook 链接，便于扩展也便于隔离敏感信息。

### 矫正 F：动手前先写 implement_plan.md

WHAT：mail.20260505T162117Z.001 在批准设计后明确要求"先出一个 implement_plan.md 执行计划，要足够具体，包括读什么 skill、从 scaffolds 复制 basic-agent 然后再之上开发、目录设计要先明确"。

WHY：设计与实现之间没有显式契约，容易在落地阶段重新发明顺序与目录布局。属于工程纪律建议。

HOW：ep.20260506T000000Z 落地 implement_plan.md（164 行 8 节），包括 Prerequisites / Starting point / Target directory layout / 12 步 Build sequence（每步 input-action-done-when-risk）/ Verification / Cron / Out-of-scope，并写入两道烟测闸门——copy.sh 后立即 probe-stream 通过才进入第 2 步，第 11 步窗口端到端跑通才进入第 12 步。

### 矫正 G：方案 A 升级 SDK pin

WHAT：第一次 copy.sh 之后 probe-stream 报 400 invalid_request_error "gpt-5.5 model requires a newer Codex"，请求 human 在三个方案里裁决，mail.20260506T022705Z.001 选定方案 A：升级 engine scaffold 的 @openai/codex-sdk pin。

WHY：basic-agent scaffold 的 SDK 还钉在 ^0.121.0，捆绑的 vendored CLI 比新 model 老。属于上游环境漂移，不是本仓代码问题；按 stop-and-report 上报正确。

HOW：ep.20260506T030000Z 把 @openai/codex-sdk 升到 ^0.128.0，重新生成 lockfile，staging-then-move 流程跑通；workdir 与 staging 两次 probe-stream 都 verdict pass。引擎脚手架 working-tree 修改未提交，等待 human 后续处置（演化为 t14）。

### 矫正 H：networkAccessEnabled true 修复 + 解耦到独立仓库

WHAT：mail.20260506T082726Z.001 同时给出两块矫正：一是 src/entry/wake-up.ts 第 72 行 startBasicThread 只传了 workingDirectory 缺 networkAccessEnabled true；二是当时 ai-informer 实现和 Builder runtime 基建混在同一个根目录下，理论上应该是独立仓库，必须迁出去。

WHY：第一项是我对 Codex SDK 选项不完整的盲点；第二项是我没有把"业务 Agent 是独立产物"这条原则提前到目录组织阶段，而是先做了端到端再发现混在一起。属于环境知识盲点叠加架构原则迟到。

HOW：ep.20260506T083000Z 同时落两件：sandbox 选项加上 networkAccessEnabled true 让 ai_sources 拿到 23 条；用 build-a-codex-agent 的 copy.sh 在 /home/ubuntu/agents/ai-informer/ 重新种了一个干净 basic-agent，把 AGENTS.md / .codex/agents / .agents/skills / ai_informer / ai_sources / config / design 整体迁过去，仅迁 Runtime/runs/delivered_dedupe_keys.txt 这条跨 run 状态。PATH A 端到端 8 分 15 秒跑通飞书广播。

### 矫正 I：仓库挪到 Builder 工作目录下 + 双 git 管理

WHAT：mail.20260506T133355Z.001 给出三件矫正：一是 Builder 旧逻辑要彻底删干净；二是 ai-informer 应该挪到 Builder 工作目录下作为开发产物；三是 Builder 与 ai-informer 都要 git 管理、未来通过 PR 改、CLAUDE.md 加入 git 管理规则、注意 .gitignore。

WHY：第一版把"独立仓库"理解成"独立目录位置"，没意识到从 Builder 视角"AI Informer 是我开发的产物"这一所有权关系应该在文件系统结构上被反映。属于所有权语义缺失。

HOW：ep.20260506T143000Z 完整 rm -rf 旧路径下的 .codex / .agents / ai_informer / ai_sources / config / dist / src / scripts / probes / node_modules / trajectories / .runs / Runtime/runs / AGENTS.md / package*.json / tsconfig.json / .gitignore；mv 把 /home/ubuntu/agents/ai-informer 整体搬到 ai-informer-builder/ai-informer/。CLAUDE.md 后续在 ep.20260506T230500Z 增补 Git management 段（双仓独立、PR-only、ai-informer/ 整子目录在 Builder .gitignore 中排除、不允许 force push 与发布后 amend）。

### 矫正 J：sandbox DNS 回归用 sandboxMode 升级

WHAT：迁移后再校验失败：sandbox 内 ai_sources 12 个源全部 gaierror -3，但宿主 shell 完全正常，networkAccessEnabled 似乎失效。mail.20260506T150344Z.001 提示"我也不知道，可能要你自己调查，或者 sandboxMode danger-full-access 试试"。

WHY：networkAccessEnabled 这一个布尔选项需要 sandboxMode workspace-write 才能真正生效——我对 Codex SDK 这一耦合关系无认知。属于第三方 SDK 隐式依赖知识盲点；同时人类也不能直接给答案，需要我自己定位。

HOW：ep.20260506T230500Z Tier 1 在 startBasicThread 选项里把 sandboxMode 设为 workspace-write，networkAccessEnabled 同时保留，npm run build 通过；PATH A 拿到 25 条候选、4 条 shortlist、4 份 briefings、飞书真实广播。后续把这条耦合关系沉淀为 heuristic--codex-sdk--sandbox-network-requires-explicit-mode.md。Tier 2 danger-full-access 没有走到。

### 矫正 K：删历史泄漏 webhook 再 push

WHAT：尝试 Builder 首次 git init 提交时，最后一道 grep 闸门发现 14 个文件历史性写过完整 webhook URL（12 份 Memory/episodes、Runtime/events.jsonl 心跳事件流、mailbox/human.jsonl 因 human 当初下发 URL）。我按 STOP 条件停下并发邮件请示走甲乙丙哪一条路。mail.20260507T010442Z.001 明确回："builder 的 webhook 相关的记录直接删了吧，不要泄露出去"。

WHY：长生命周期 agent 通过有日志通道接收的 secret 会扩散到 mailbox、events、episode 等多份持久化记录，这一扩散面在第一版没被建模。属于安全模型缺失。还有附带问题：执行 sed 时如果 grep / sed 命令本身出现字面 webhook URL，会被心跳日志再次写入 events.jsonl，造成"删了又补回"的失败模式。

HOW：ep.20260507T011000Z 用 shell 变量绑定 secret 模式（PATTERN_URL=$(printf 'https://...%s' '<token>') 拼出后从不再字面出现），把 sed → JSON 校验 → git add → git grep 验证 → commit 串成单次无让出 shell 调用。15 个文件全部置换为 <FEISHU_WEBHOOK_REDACTED> 与 <FEISHU_TOKEN_REDACTED>，Builder 仓库 commit 712b2e5 推送成功。这条经验沉淀为 heuristic--secret-redaction--single-burst-in-heartbeat-logged-workspace.md 与 heuristic--long-running-agent-secrets--accumulate-in-records.md 两份笔记，详见 ep.20260507T013930Z。

### 次要修正（一组）

下列是构建过程中由 planner 或 evaluator 而非 human 直接发起的小幅修正，列 WHAT 即可：

- recommendation-rendering 在 implement_plan 与 todo 描述里仍残留旧名称（在矫正 B 之后），ep.20260506T050000Z 顺手重写为符合当前设计的自然语言措辞。
- design §3 stripped-line 描述与 design/subagents/screener.md stripped-line 例子在 source_categories 字段上不一致（一个含一个不含），ep.20260505T143000Z 在 reflection 里登记，由 ep.20260506T060000Z 在写 helper 脚本时一次性对齐。
- 子代理 .toml 的 allowed-tools / sandbox_mode 字段格式不能照抄 Claude-Code SKILL.md frontmatter 风格，需用 Codex TOML schema；ep.20260506T050000Z 的 evaluator 现场更正。
- COMPLETION_AUDIT.md 行数裁剪：planner 给的 ≤200 上限实际用了 104 行就讲清四节，无需扩写凑数。
- "URL containment 是否覆盖 episode 正文"在 ep.20260505T110000Z reflection 里第一次被识别，但第三占位（mailbox/Runtime events 也累积）直到 ep.20260507T011000Z 被 human 矫正才补全。

## 第二节：瘦身评估

下面分两节：先列已无需的旧逻辑/脚手架，再列可优化的目录结构。本节不执行任何动作。

### 2.1 已无需的旧逻辑/脚手架

- design/ 与 ai-informer/design/ 双份。Builder 工作目录根的 design/ 是设计阶段产物，迁仓时整树复制到 ai-informer/design/ 后没有删除原件。两份内容已经 diff 等价（diff -rq 无差异），保留 ai-informer/design/ 即可，Builder 工作目录的 design/ 可降级为只读引用或直接删除——它已经完成了设计阶段的承载使命。
- /home/ubuntu/agents/builders/ai-informer-builder/implement_plan.md 与 /home/ubuntu/agents/builders/ai-informer-builder/COMPLETION_AUDIT.md。前者是 t2 子任务 2-9 的实施合同，所有 12 步在 ep.20260506T060000Z 之前已完结；后者是 PATH A 第一次失败时段的状态拍照（status: PATH A failed），PATH A 后来已在 ep.20260506T230500Z 跑通且 cron 已上线，整份审计的"DONE / PENDING / RECOMMENDATIONS"对当前生产态都已过期。两份文件作为构建史可保留在仓库中，但应在文首加 archived 标注或迁入 design/archive/。
- 引擎脚手架共享 working-tree 三件改动（/home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent/ 下的 package.json、package-lock.json、src/entry/wake-up.ts），分别承载 SDK pin 升级（^0.121.0 → ^0.128.0）与 networkAccessEnabled + sandboxMode workspace-write 修复。它们目前以未提交工作树形式存在，下游任何新 copy.sh seed 都会拿不到这些修复。t14 已记账等待 human 决断走"PR 回流上游 vs 本地 fork 标注"。在尘埃落定前，这三处改动既不可回退也不可遗忘。
- ai-informer/.runs/ 与 ai-informer/trajectories/ 累积。每次 PATH A 跑都会产出 .runs/<runId>/ 与 trajectories/<runId>.jsonl 两份记录；当前已沉积两份（2026-05-06T13-41-52-083Z-wixyl4 失败、2026-05-06T15-11-11-250Z-46m9q3 成功），未来每天 3 次 cron 触发会持续累积。.gitignore 已经排除二者，但磁盘清理没有回收策略。建议加一条按 N 天保留窗口或按 success/failure 选择性保留的策略。
- ai-informer/scripts/cron.example 与生产 crontab 不一致。文件内容仍是 basic-agent scaffold 默认（每 15 分钟 dist/entry/scheduled.js），主机实际安装的是 0 8/12/18 三条 ai-informer/scripts/run-once.sh。ep.20260506T060000Z 因 PATH A 失败跳过该步骤，后续 cron 实际安装时 cron.example 没有被同步更新。建议把 cron.example 改写为三条 0 HH 窗口示例并把 wake-up message 模板写进去。
- ai-informer/probes/ 下保留了 basic-agent 自带的 probe-skills.mjs 与 probe-stream.mjs。在 build-a-codex-agent SKILL.md 文档化的语境下二者是 copy.sh 之后立刻烟测使用的；构建期已两次发挥作用（ep.20260506T010000Z 与 ep.20260506T030000Z）。生产期它们不再被自动调用，但在引擎脚手架升级时仍可作为快速烟测工具，可保留但建议在 README 里降级为"运维自检脚本"。
- ai-informer/.codex/agents/ 下保留了 README.md 与 web_researcher.toml.example 两份脚手架原件（来自 basic-agent 出厂模板）。AI Informer 自身只用到 screener.toml 与 researcher.toml；前两份从未被调用、内容是泛化示例。可以删除以避免读者混淆，或显式归入 .codex/agents/_examples/。

### 2.2 可优化的目录结构

- ai-informer 内嵌套 .runs/ 包装与 Runtime/runs/ 规范名重复。Codex SDK 把每次运行的 cwd 设为 .runs/<runId>/，而 design §3 与 AGENTS.md 的契约用的是 Runtime/runs/<window>/<YYYYMMDD-HHMM>/。结果是 PATH A run 实际生产 .runs/<runId>/Runtime/runs/<NN>/<TS>/ 这种双层嵌套；跨 run 状态 delivered_dedupe_keys.txt 在 sandbox 里被认为只读、需要写到 .runs 副本里再合并，这一耦合在 ep.20260506T230500Z 与 ep.20260507T011000Z 都报告过 t10 仍开放。建议二选一：要么把 Codex --cd 切成 workdir 根使 Runtime/runs/ 成为 sandbox 内规范路径；要么把契约直接改成 .runs/<runId>/Runtime/runs/<NN>/<TS>/ 接受现实并放弃顶层 Runtime/runs/。
- Builder 工作目录顶层混合了行为状态（CLAUDE.md / .claude / Memory / mailbox / Runtime / todo_list）与构建期实施记录（implement_plan.md / COMPLETION_AUDIT.md / design/）。前者是 Builder 自身行为不可移除，后者是一次性产物。可以建立 design/archive/ 或 build-records/ 子目录把 implement_plan.md / COMPLETION_AUDIT.md / RETROSPECTIVE_v1.md（本文件）等里程碑文件归拢，让顶层只露 Builder 自身行为状态。
- ai-informer 同时混合 Python（ai_informer/ 通知模块、ai_sources/ 数据来源层）与 TypeScript（src/、dist/、node_modules/、package*.json、tsconfig.json）在同一层级，对新入仓阅读者认知负担偏高。可以引入 python/ 与 typescript/ 顶层分组，或用 README 上的 Reading Order 段把"先读 AGENTS.md → 再读 .agents/skills → 再看 ai_informer/notifications → TS 部分仅作为 Codex SDK 装载机不需常读"的导航线明确出来。
- Memory/episodes/2026/05/ 已积累 23 份 episode 文件并仍在持续增长。将来跨月/跨年浏览时按文件名扫读已经吃力；planner 索引 episode 也得每次列目录。可以在每月末或每达到 25 份时生成一份 month-index.md（含 id / title / status / 主要矫正点），新增 episode 时由 advanced-episode-flow 自动 append 一行。
- design/ 双份位置（详见 2.1 条 1）属于结构问题不只是冗余问题。设计阶段产物的归属应在工程开始时就明确——它属于业务 agent（ai-informer/design/）还是属于 Builder 的设计记录（builder/design/）需要二选一。本次留在 ai-informer/design/ 更符合"AI Informer 是产物"的所有权原则，Builder 顶层 design/ 删除即可使结构清晰。

## 结语

AI Informer v1 的构建轨迹包含 11 次显式 human 矫正、3 次执行受阻停步上报、4 次设计层重写。整体留下的耐用经验是：业务 Agent 与 Builder 自身运行框架的词汇必须严格隔离；LLM 子代理用自然语言契约而不用 strict-JSON 契约；中间产物上下文不进 LLM 而走文件介导；secret 在长生命周期 agent 里会通过日志通道扩散，必须建模。这些经验已沉淀进 5 份 heuristic 笔记与 CLAUDE.md 的 Git management 段落。本回顾不修改任何其他文件，瘦身建议留待 human 审阅再分别开 episode 执行。

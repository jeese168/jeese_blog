# Superpowers 工作流指南 2.0  
  
这份文档是对现有工作流说明的 2.0 版整理。目标不是发明一套新的说法，而是尽量忠于 Superpowers 当前 skills 的原意，同时保留便于人理解、便于口头复述的表达方式。  
  
一句话概括：  
  
**Superpowers 不是一个可视化工作流平台，而是一套通过 skills 装载进 Agent 的工程纪律系统。**  
  
它不靠网页节点图来强制流程，而是靠一组 skill 把下面这些东西写进 Agent 的行为里：  
  
- 先判断该用哪个 skill  
- 先设计，再计划，再实现  
- 先证据，再结论  
- 先 failing test，再写实现  
- 先审查，再进入下一步  
  
所以它的思想不是“让模型自由发挥”，而是“让模型像一个守流程的工程团队那样工作”。  
  
---  
  
## 先讲总原则  
  
### 1. `using-superpowers` 是入口 skill，不是业务 skill  
  
它的作用不是直接帮你写代码，而是先把规矩立住：  
  
- 任何用户请求进来，先判断有没有相关 skill  
- 哪怕只有 1% 可能适用，也先加载 skill 再行动  
- skill 检查发生在回答之前，甚至发生在澄清问题之前  
  
简单理解：  
  
**它像一个总控路由器。它不亲自做实现，而是决定接下来应该把任务送到哪条 skill 链上。**  
  
### 2. 用户要求优先，skill 次之  
  
Superpowers 很强势，但它不是最高优先级。当前用户的明确要求、项目里的 `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` 这类文件，优先级还是更高。  
  
所以更准确的说法不是“skill 永远覆盖一切”，而是：  
  
**只要用户没明确推翻某条 workflow，Superpowers 就会要求 Agent 遵守它。**  
  
### 3. 它不是“一个 skill 做到底”，而是“多个 skill 组成阶段门”  
  
这套系统最像的不是一个万能提示词，而是一串阶段门：  
  
- 先决定是否进入设计阶段  
- 设计通过后，才能写 spec  
- spec 通过后，才能写 plan  
- plan 准备好后，才能进入执行  
- 执行中每个任务都要验证、审查  
- 全部完成后，才能决定 merge / PR / 保留 / 丢弃  
  
所以它本质上是：  
  
**一个由多个 skills 组成的大工作流图，而不是一个平铺的提示词集合。**  
  
---  
  
## 场景一：从零构建新项目（0→1）  
  
这是最完整、最能体现 Superpowers 思想的路线。  
  
```text  
using-superpowers（会话入口）  
    → brainstorming（把想法变成 design / spec）  
        → using-git-worktrees（准备隔离工作区）  
            → writing-plans（把 spec 拆成执行计划）  
                → subagent-driven-development 或 executing-plans（执行）  
                    → test-driven-development（实现时的硬纪律）  
                    → review / verification（审查与验证）  
                        → finishing-a-development-branch（收尾）  
```  
  
### 这条路线背后的思想  
  
它不是“想法来了就开始写”，而是：  
  
1. 先把需求讲清楚  
2. 再把设计写清楚  
3. 再把实施顺序写清楚  
4. 再进入实现  
5. 实现过程中不能跳过测试和审查  
6. 做完以后不能自己随便决定怎么收尾  
  
换句话说：  
  
**Superpowers 认为，好的实现不是从编码开始的，而是从约束开始的。**  
  
### `brainstorming` 阶段  
  
这个阶段的目标是：  
  
**把“一个模糊想法”变成“一个可批准的设计/spec”。**  
  
它要求做的事情是：  
  
- 先探索当前项目上下文，读文件、文档、最近提交  
- 如果后面会涉及视觉问题，可以单独询问是否启用 visual companion  
- 每次只问一个澄清问题，不允许一条消息里堆很多问题  
- 提出 2-3 个可选方案，讲清 trade-off 和推荐理由  
- 分节展示设计，让用户逐节确认  
- 写出 spec 文档，提交 git  
- 自审 spec，扫掉占位符、矛盾、歧义、范围失控  
- 明确停下来，请用户审阅 spec 文件  
- 用户批准后，才进入 `writing-plans`  
  
#### 硬性禁止  
  
在用户批准设计之前：  
  
- 不许写代码  
- 不许脚手架  
- 不许调用实现类 skill  
- 不许偷偷跳到 plan 或实现  
  
这不是建议，是明确的 gate。  
  
#### `spec` 在这里到底是什么  
  
`spec` 不是任务列表，也不是 commit checklist。  
  
它回答的是：  
  
- 你到底要做什么  
- 为什么这么做  
- 范围边界在哪里  
- 架构和数据流怎么设计  
- 出错时怎么处理  
- 怎么测试这件事  
  
所以 `spec` 的本质是：  
  
**面向“设计正确性”的文档。**  
  
### `using-git-worktrees` 阶段  
  
设计批准之后，不是立刻写 plan，而是先准备隔离工作区。  
  
这一步的目的不是炫技，而是为了避免：  
  
- 在主工作区里把未完成 feature 搅进当前分支  
- 把本来不相关的改动混到一起  
- 后面很难 clean up  
  
skill 的做法是：  
  
- 先检查项目里是否已经有 `.worktrees/` 或 `worktrees/`  
- 如果没有，再看 `CLAUDE.md`  
- 还没有，就问用户放哪  
- 如果是项目内目录，必须先确认目录被 `.gitignore` 忽略  
- 然后创建 worktree、新分支、自动跑项目安装/初始化  
- 再做一次基线验证，确认当前 worktree 一开始就是干净的  
  
#### 如果基线测试失败  
  
不能装作没看到继续做。  
  
skill 要求：  
  
- 报告失败  
- 给用户看现在的状态  
- 问用户是继续、还是先调查基线问题  
  
这里体现出的思想是：  
  
**先确认出发点是干净的，否则你后面连“新 bug 是谁引入的”都分不清。**  
  
### `writing-plans` 阶段  
  
这一步不是再解释一遍设计，而是把设计拆成实现顺序。  
  
`plan` 回答的是：  
  
- 先做哪一步  
- 改哪些文件  
- 每一步的测试怎么写  
- 每一步怎么验证  
- 每一步完成后怎么提交  
  
所以 `plan` 的本质是：  
  
**面向“执行正确性”的文档。**  
  
#### 它要求的颗粒度  
  
每个任务都要拆到 2-5 分钟一级，理想形态是这种节奏：  
  
1. 写 failing test  
2. 跑一下确认它真的 fail  
3. 写最小实现  
4. 再跑确认 pass  
5. 提交  
  
而且每一步要尽量给出：  
  
- 精确文件路径  
- 具体命令  
- 预期输出  
- 需要写的代码  
  
#### 它不允许的写法  
  
这种写法都属于 plan 失败：  
  
- `TODO`  
- `TBD`  
- “添加适当的错误处理”  
- “参考上面的实现”  
- “这里自行补测试”  
  
也就是说：  
  
**plan 不是提纲，而是可以直接交给执行者照着做的操作说明。**  
  
#### `spec` 和 `plan` 的最短区分  
  
- `spec`：你要建什么，为什么这样建  
- `plan`：你准备怎么一步步把它建出来  
  
如果再压缩成一句：  
  
**spec 管设计，plan 管施工。**  
  
#### 关于 plan 的审批  
  
它不像 spec 那样有一个非常明确的“停下来请你审阅文档再往前走”的硬门。  
  
更接近的流程是：  
  
- 写完 plan  
- 自审 plan  
- 然后问用户选哪种执行方式  
  
所以实操里你可以把它理解成：  
  
**spec 的审批门比 plan 更明确；plan 的放行更多体现在“用户选择开始执行”。**  
  
### 执行阶段：`subagent-driven-development` 或 `executing-plans`  
  
写完 plan 后，Superpowers 不只提供一种执行方式。  
  
#### 执行阶段的整体结构：总—分—总  
  
在看具体路线之前，先把执行阶段的整体结构讲清楚，否则很容易搞混 TDD、review、verification 这几个东西之间的关系。  
  
执行阶段是**总—分—总**的结构：  
  
```text  
【总】spec 确定要做什么  
  
【分】plan 拆出 N 个任务，每个任务串行执行：  
    Task 1：  
        implementer（TDD：先写 failing test → 写实现 → 看 pass）  
            → spec reviewer（这个任务做的和要求的一样吗）  
                → code quality reviewer（代码质量够不够好）  
                    → 两道审查都通过，进入 Task 2    Task 2：同上  
    Task 3：同上  
    ...  
【总】所有任务完成后，跑完整测试套件做全局 verification    → 捕获跨任务的回归（Task 5 有没有破坏 Task 2 的行为）  
    → 拿到当场运行的新鲜证据，才能声称整个 plan 完成  
```  
  
几个容易混的点：  
  
- **任务之间是串行的，不是并行的。** skill 明确禁止同时派多个实现子 Agent，否则会产生代码冲突。  
- **TDD 是任务内部的纪律**，每个任务自己写 failing test、实现、验证通过。它保证的是”这一步做对了”。  
- **全局 verification 是跨任务的**，跑的虽然是同一套测试，但目的不同——它保证的是”所有步骤加在一起还是对的”，并且要求有当场运行的输出作为证据。  
  
#### 路线 A：`subagent-driven-development`  
  
这是更能体现”多角色工程团队”思想的路线。  
  
主会话负责：  
  
- 读 plan  
- 提取任务  
- 给每个子 Agent 组织上下文  
- 协调审查和修复  
  
子 Agent 则按任务执行。  
  
最关键的一点是：  
  
**它不是一个实现 Agent 一路跑到底，而是每个任务都可能经历多角色链路。**  
  
典型链路是：  
  
```text  
主控 Agent    → implementer subagent（实现）  
        → spec reviewer subagent（查是否符合 plan / spec）  
            → code quality reviewer subagent（查代码质量）  
                → 通过后进入下一个任务  
```  
  
#### 两层 review 是什么含义  
  
- 第一层 review 看的是：做的东西是不是和要求的一样  
- 第二层 review 看的是：即便方向对了，代码质量够不够好  
  
这两层分开的意义在于：  
  
**“做对” 和 “写好” 是两件不同的事。**  
  
#### 如果 implementer 说自己卡住了  
  
不能简单粗暴地让它“再试一次”。  
  
skill 要求主控去判断：  
  
- 是不是上下文不够  
- 是不是模型能力不够  
- 是不是任务太大  
- 是不是 plan 本身就有问题  
  
然后根据原因补上下文、换模型、拆任务，或者回到用户这里。  
  
#### 模型选择原则  
  
上游 skill 写的是能力层级，不是具体品牌型号：  
  
- 机械、局部、边界清晰的任务：用便宜且快的模型  
- 多文件集成、需要判断的任务：用标准模型  
- 架构、设计、全局审查：用最强模型  
  
所以这里更忠于原文的说法不是 “Haiku / Sonnet / Opus”，而是：  
  
**cheap / standard / most capable 三档。**  
  
#### 路线 B：`executing-plans`  
  
这条路是 plan 已经有了，但不用那种“每个任务一个 fresh subagent”的强编排。  
  
它的特点是：  
  
- 先读 plan  
- 批判性审查 plan  
- 发现问题先问用户，不强冲  
- 没问题再逐任务执行  
- 执行完再进入 `finishing-a-development-branch`  
  
它仍然是守流程的，只是没有 `subagent-driven-development` 那么强调子 Agent 协作。  
  
简单讲：  
  
- `subagent-driven-development` 更像小型工程团队协同  
- `executing-plans` 更像一个主 Agent 按计划执行，但仍然守 checkpoint  
  
### `test-driven-development` 在执行阶段里的角色  
  
TDD 不是一个“可有可无的好习惯”，而是这套系统里最硬的纪律之一。  
  
它的铁律是：  
  
**没有先看到 failing test，就不允许写 production code。**  
  
对 Superpowers 来说，下面这些场景都默认要用 TDD：  
  
- 新功能  
- bugfix  
- refactor  
- 行为变更  
  
所以实现阶段不能理解成“先写，再测”。  
  
更准确的理解是：  
  
**写测试、看它失败、写最小实现、看它通过，这本身就是实现步骤的一部分。**  
  
### review 和 verification 在执行阶段里的角色  
  
实现不是做完代码就结束。  
  
这套系统至少还要求两类后置约束：  
  
#### 1. review  
  
review 的作用是防止：  
  
- 偏离 spec  
- 代码质量不够  
- 带着明显问题进入下一任务  
  
README 里把 `requesting-code-review` 也列进基础工作流，所以你可以把它理解成：  
  
**review 不是额外加分项，而是这套方法论的一部分。**  
  
#### 2. verification  
  
`verification-before-completion` 的核心不是“再检查一下”，而是：  
  
**没有新鲜证据，就不能声称完成。**  
  
比如你要说“测试都过了”，那你得先真跑一遍测试命令，并看输出。  
  
它防的是这种很常见的 Agent 失败：  
  
- 没跑就说通过了  
- 只看了一部分输出就说没问题  
- 改了代码就默认 bug 修好了  
  
### `finishing-a-development-branch` 阶段  
  
所有任务都完成并验证后，才进入收尾。  
  
它要求做的事很明确：  
  
1. 先再次验证测试通过  
2. 再确定基线分支  
3. 然后给用户 4 个固定选项  
  
固定选项是：  
  
4. 本地合并回基线分支  
5. 推送并创建 PR  
6. 暂时保留分支和 worktree  
7. 丢弃这次工作  
  
#### 这一步的思想  
  
它不允许 Agent 自己拍脑袋决定：  
  
- 是 merge 还是开 PR  
- 是保留还是删除 worktree  
- 是不是应该直接丢弃  
  
也就是说：  
  
**Superpowers 把“怎么收尾”也设计成了一个显式决策点，而不是隐式默认行为。**  
  
#### 关于 PR 场景下是否清理 worktree  
  
这里上游 skill 文本本身有一处细节歧义：  
  
- 正文里有一处写法看起来像 Option 2 之后也会进入 cleanup  
- 但它的 Quick Reference 和 Red Flags 又更像是在说：Option 2 通常保留 worktree  
  
如果按更保守、也更符合常见工作习惯的理解：  
  
- Option 1：合并后清理  
- Option 2：创建 PR 后通常保留  
- Option 3：保留  
- Option 4：丢弃后清理  
  
这也是本文采用的理解。  
  
---  
  
## 场景二：在现有代码上加新需求  
  
主路线和 0→1 一样，区别不在“走不走这些阶段”，而在每个阶段怎么理解“现有代码”。  
  
```text  
using-superpowers  
    → brainstorming（先理解现有项目和新增需求）  
        → using-git-worktrees            → writing-plans                → subagent-driven-development 或 executing-plans                    → TDD / review / verification                        → finishing-a-development-branch  
```  
  
### 这个场景里的核心思想  
  
不是从零设计系统，而是：  
  
**先理解现有边界，再决定新需求应该怎样接进去。**  
  
### `brainstorming` 在这个场景里的重点  
  
它不是直接问“你想要什么功能”，而是会先看：  
  
- 当前项目结构  
- 现有模块职责  
- 已有接口模式  
- 命名和组织习惯  
  
然后再问：  
  
- 新需求挂在哪一层  
- 是否复用已有接口  
- 是纯新增还是要扩展现有边界  
  
#### 关于顺手重构  
  
它的原则不是“见乱就重构”，而是：  
  
- 和当前需求直接相关的结构问题，可以纳入设计  
- 不相关的历史包袱，不要偷偷夹带进这次 spec  
  
所以它想防的是这种事：  
  
**用户明明只是想加一个搜索功能，最后 spec 却变成了全项目重构计划。**  
  
### `writing-plans` 在这个场景里的重点  
  
计划里不再只是“新建哪些文件”，而会更多地写：  
  
- 修改哪个现有文件  
- 改哪一类职责  
- 哪些调用方要跟着调整  
- 哪些现有测试要回归  
  
所以在这个场景里：  
  
**plan 的重点不只是“新增什么”，而是“如何安全地改旧系统”。**  
  
### 执行阶段的重点  
  
子 Agent / 执行者拿到的上下文，不应该是“你自己去仓库里瞎找”。  
  
主控更理想的做法是：  
  
- 把相关文件  
- 对应 plan 片段  
- 需要遵守的边界  
- 验证要求  
  
一并交给它。  
  
这样做的思想其实很清楚：  
  
**Controller 负责整理上下文，Implementer 负责消化并执行。**  
  
---  
  
## 场景三：重构  
  
重构不是一个单一路线，它要先看这次改动到底有没有设计判断。  
  
### 路线 A：机械重构  
  
例如：  
  
- 统一命名风格  
- 抽出常量  
- 去掉重复字面量  
- 纯粹的结构整理，不改变外部行为  
  
这种情况下，不一定非要走完整的 `brainstorming -> spec -> plan`。  
  
但这不等于“随手改一把就行”。  
  
#### 机械重构至少仍然要守的纪律  
  
- 先经过 `using-superpowers` 的 skill 检查  
- 遵守 `test-driven-development` 对 refactoring 的要求  
- 做完整验证  
- 不能把重构顺手变成加功能  
  
简单说：  
  
**可以不一定先写 spec，但不能不守测试和验证纪律。**  
  
### 路线 B：有设计判断的重构  
  
例如：  
  
- 模块边界要重划  
- 职责要拆分  
- 耦合关系要解开  
- 对外接口语义要重新梳理  
  
这时就不是机械整理了，而是设计问题。  
  
路线会回到完整链路：  
  
```text  
using-superpowers  
    → brainstorming        → using-git-worktrees            → writing-plans                → subagent-driven-development 或 executing-plans                    → TDD / review / verification                        → finishing-a-development-branch  
```  
  
#### 这个场景的核心思想  
  
Superpowers 反对的是：  
  
**拿“重构”这个词当借口，把没有想清楚边界的改动直接推进代码里。**  
  
所以只要重构涉及设计判断，它就会把你重新拉回 spec 和 plan。  
  
---  
  
## 场景四：修 bug  
  
修 bug 的入口不是 `brainstorming`，而是专用的诊断 skill。  
  
```text  
using-superpowers  
    → systematic-debugging        → [简单且边界清晰的修复：进入 TDD + 实施 + verification]        → [复杂修复：writing-plans → subagent-driven-development/executing-plans]  
```  
  
### `systematic-debugging` 阶段  
  
这个 skill 的中心思想非常硬：  
  
**没有根因调查，就不允许提出修复。**  
  
它反对的不是”速度”，而是”猜”。  
  
#### 四个阶段，必须按顺序走完  
  
**Phase 1：根因调查**  
  
在提出任何修复之前，必须完成：  
  
1. 认真读报错信息——不要跳过，stack trace 要完整看  
2. 稳定复现问题——能不能可靠触发？不能复现就继续收数据，不猜  
3. 看最近的代码变更——git diff、最近提交、新依赖、配置变化  
4. 在多组件系统中补诊断信息——在每个组件边界加日志，跑一次拿到证据，再分析是哪层出了问题  
5. 追踪数据流——坏的值从哪里来？一层一层往上追，找到源头，在源头修，不在症状处修  
  
**Phase 2：模式分析**  
  
6. 找现有的可工作的类似代码  
7. 和参考实现完整对比，不要跳过任何一行  
8. 列出所有差异，不管看起来多不重要  
  
**Phase 3：假设验证**  
  
9. 提出一个明确的单一假设：”我认为根因是 X，因为 Y”  
10. 用最小改动验证这个假设——一次只改一个变量  
11. 验证通过才进入 Phase 4；不通过则重新形成新假设，不在旧假设上叠加修改  
  
**Phase 4：实现**  
  
根因确认之后，才做这些：  
  
12. **写 failing test**——最简单的能复现问题的测试用例，必须先有，再动代码  
13. 针对根因做单一修复，不顺手改其他东西  
14. 验证 test pass，验证其他测试没有被破坏  
  
#### 如果 3 次修复都失败了  
  
skill 明确要求停下来，不能再提第 4 个 fix。  
  
3 次失败通常意味着这不是一个局部 bug，而是架构问题。需要和用户讨论是否应该重新审视设计，而不是继续在症状上打补丁。  
  
### 修 bug 时和 TDD 的关系  
  
TDD 出现在 Phase 4 的第一步——**failing test 是在根因确认之后才写的**，不是诊断的起点。  
  
它的作用是：把已经确认的根因固化成一个可验证的测试，再基于这个测试做修复。  
  
**debugging 负责找到根因，TDD 负责把修复落成可信代码，全局 verification 确认修复没有破坏其他东西。**  
  
### 简单修复和复杂修复的分叉  
  
根因清楚、改动边界小的（一两个文件、不改接口语义）：  
  
- 直接在 Phase 4 里完成，不需要单独写 plan  
  
改动涉及多文件、接口变更、跨模块连锁影响的：  
  
- 升格为走 writing-plans，再进入执行阶段  
- 跳过 brainstorming 和 spec，但执行和验证的纪律和正常流程一样  
  
### 修 bug 场景的核心思想  
  
Superpowers 想防止的是：  
  
- 看到报错就猜，没有第一手证据  
- 多个 fix 一起上，搞不清是哪个改动生效的  
- 修的是症状不是根因  
- 改完没验证就宣布结束  
  
**先读错误，再复现，再收证据，再分析模式，再假设，再最小验证，根因确认后才写 failing test 和修复。**  
  
---  
  
## 各场景路线对照  
  
| 场景 | 入口 | 是否需要 brainstorming | 是否需要 writing-plans | 绝不能跳过的东西 |  
|------|------|----------------------|----------------------|------------------|  
| 0→1 新项目 | `using-superpowers` → `brainstorming` | 是 | 是 | worktree、TDD、review、verification |  
| 加新功能 | `using-superpowers` → `brainstorming` | 是 | 是 | 理解现有边界、TDD、review、verification |  
| 复杂重构 | `using-superpowers` → `brainstorming` | 是 | 是 | 边界设计、TDD、review、verification |  
| 机械重构 | `using-superpowers` → 相关 skill | 不一定 | 不一定 | TDD、verification |  
| 修 bug | `using-superpowers` → `systematic-debugging` | 否 | 视复杂度 | 根因调查、TDD、verification |  
  
---  
  
## 把这套方法再压缩成一句人话  
  
Superpowers 的思想不是：  
  
“给 Agent 一堆提示词，让它表现更聪明。”  
  
而是：  
  
**把软件工程里那些本来应该由一个成熟团队遵守的纪律，拆成可装载的 skills，让 Agent 在每个阶段都被这些纪律约束。**  
  
它的价值不在于某一个 skill 特别神，而在于这些 skill 连起来之后，会不断把 Agent 拉回下面这条主线：  
  
```text  
先判断流程  
    → 先设计  
        → 再计划  
            → 再隔离环境  
                → 再执行  
                    → 执行中必须测试和审查  
                        → 完成前必须验证  
                            → 最后才决定怎么收尾  
```  
  
所以如果你要用一句最容易复述的话来讲它，可以这么说：  
  
**Superpowers 不是一个“会做事的 Agent 功能包”，而是一套“让 Agent 按工程纪律做事”的技能化工作流系统。**
# OpenSpec 内网 Linux 入门使用文档

适用版本：**v1.15.0**。核实日期：2026-10-26。

适用条件：只能下载 GitHub 仓库和 Release；不希望手动构建；下载后可以把文件放到 Linux；已有能读取文件、执行 Shell 的内部 Coding Agent。

**推荐路线：下载官方预编译 Linux 程序和 SHA256SUMS → 复制到 Linux → 校验并赋予执行权限 → 关闭联网检查 → 初始化通用技能 → 让内部 Agent 读取技能文件开展工作。**

这条路线不需要安装 Node.js、npm、pnpm 或 Bun，也不需要编译 OpenSpec。Linux 的 CPU 架构和基础运行库仍须与下载文件匹配。

## 1. 先知道 OpenSpec 负责什么

OpenSpec 把一次变更组织成需求、设计、规格和任务文件。它的 CLI 管目录、状态、模板、规格检查和归档；你的内部 Agent 负责理解需求、编写文档、修改代码和调用工具。

| 入口 | 在哪里使用 | 示例 |
|---|---|---|
| `openspec ...` | Linux Shell | `openspec init`、`openspec validate` |
| `/opsx:...` 或技能名称 | 支持相应集成的 AI 对话框 | `/opsx:propose`，具体拼写随客户端变化 |
| “读取指定 SKILL.md 并执行” | 你的内部 AI 对话或非交互任务输入 | 本文推荐的通用用法，无需假定快捷命令已受支持 |

安装 OpenSpec 不会安装模型，也不会替换内部 CLI。使用它的本地规格功能，不需要申请 OpenSpec API Key。

## 2. 应该下载哪些文件

### 2.1 固定使用 v1.15.0

打开官方页面：

[OpenSpec v1.15.0 Release](https://github.com/Fission-AI/OpenSpec/releases/tag/v1.15.0)

在 **Assets** 中下载：

1. 与目标 Linux 匹配的一个程序文件；
2. 同一版本的 **SHA256SUMS**。

**不要把 `Source code (zip)` 或 `Source code (tar.gz)` 当成免构建安装包。** 它们是源码；本次免构建安装使用下表中的程序文件。

| 目标 Linux | 下载的准确文件名 | 直接下载 |
|---|---|---|
| x86_64，glibc 2.17+，常见服务器/PC Linux | `openspec-linux-x64` | [下载](https://github.com/Fission-AI/OpenSpec/releases/download/v1.15.0/openspec-linux-x64) |
| aarch64 / arm64，glibc 2.17+ | `openspec-linux-arm64` | [下载](https://github.com/Fission-AI/OpenSpec/releases/download/v1.15.0/openspec-linux-arm64) |
| x86_64，musl，例如 Alpine 3.17+ | `openspec-linux-x64-musl` | [下载](https://github.com/Fission-AI/OpenSpec/releases/download/v1.15.0/openspec-linux-x64-musl) |
| aarch64 / arm64，musl，例如 Alpine 3.17+ | `openspec-linux-arm64-musl` | [下载](https://github.com/Fission-AI/OpenSpec/releases/download/v1.15.0/openspec-linux-arm64-musl) |
| 对应版本的校验文件 | `SHA256SUMS` | [下载](https://github.com/Fission-AI/OpenSpec/releases/download/v1.15.0/SHA256SUMS) |

官方说明：musl 版本需要目标系统具备 `libstdc++` 和 `libgcc` 运行库。这里的版本要求来自官方安装文档；本文实际执行验证覆盖 x86_64 + glibc 2.39。

下载可以在 Windows、macOS 或另一台能访问 GitHub 的电脑上完成。之后按你允许的方式传到目标 Linux；不要求目标 Linux 自己访问 GitHub。

如果方便，也可以另带一份 [v1.15.0 源码 ZIP](https://github.com/Fission-AI/OpenSpec/archive/refs/tags/v1.15.0.zip)，用于离线查看 `docs/`，不需要对它执行构建。

### 2.2 如何选对 Linux 文件

在目标 Linux 上执行：

```bash
uname -s
uname -m
getconf GNU_LIBC_VERSION
```

常见结果：

```text
Linux
x86_64
glibc 2.31
```

这种情况下载 `openspec-linux-x64`。

如果 `getconf` 不存在或不能返回 glibc 信息，再检查：

```bash
ldd --version
```

输出表明是 musl 才选择 `-musl` 文件；不要仅凭 `getconf` 报错就确定 libc 类型。x86_64 和 arm64 也不能混用。

如果现在无法查看目标 Linux，且只方便下载一次，可以把四个 Linux 程序和 SHA256SUMS 全部带上，到目标机再选择。普通 `linux-x64` 文件本次下载大小约 89 MiB。

只有“Linux”这个条件，不能保证任何二进制都能运行；ARM32、MIPS 等不在这份 Release 的 Linux 平台列表中。

## 3. 复制到 Linux 并安装

以下以 `openspec-linux-x64` 为例。若你下载的是其他架构，只修改 `openspec_asset` 那一行。

### 3.1 校验下载文件

把程序和 SHA256SUMS 放在同一个目录，先进入该目录。**校验前保留原始文件名。**

```bash
cd /你的实际下载目录

openspec_asset=openspec-linux-x64
awk -v f="$openspec_asset" '$2 == f { print }' SHA256SUMS | sha256sum -c -
```

预期输出：

```text
openspec-linux-x64: OK
```

只校验所选文件即可。直接检查完整 SHA256SUMS 会尝试查找 Windows/macOS 等其他程序，导致“找不到文件”的提示。

如果不是 `OK`，先停止安装，检查文件名和下载是否完整，再从同一个 Release 重新下载。

本次实测的 v1.15.0 x64/glibc 程序 SHA256 为：

```text
15dbc91d6c0c8bedda936043f2fc0ea6ce6ca8011e6a40273cb9f5c586cff564
```

实际安装仍以你下载的同版本 SHA256SUMS 为核对依据。

### 3.2 安装到用户目录，无需 sudo

确认上一步校验通过后执行：

```bash
mkdir -p "$HOME/.local/bin"
install -m 0755 "$openspec_asset" "$HOME/.local/bin/openspec"
export PATH="$HOME/.local/bin:$PATH"
```

这只是复制一个可执行文件。不要运行源码构建，也不用执行任何在线包管理器安装命令。

### 3.3 首次运行前，关闭后台外联检查

在启动 OpenSpec 和内部 Agent 的同一终端中执行：

```bash
export OPENSPEC_TELEMETRY=0
export DO_NOT_TRACK=1
export OPENSPEC_NO_UPDATE_CHECK=1
```

含义：关闭 OpenSpec 使用统计，声明不追踪，并关闭版本检查。变量要在首次调用前设置。

这些设置不等于整个 Agent 或其他 MCP 的网络隔离；其他组件仍沿用公司原有内网配置。本文流程不使用反馈提交、在线安装、远程 store 或自动下载升级。

每个新终端都需要有这些变量。确认有效后，可以按自己的 Shell 管理习惯把 PATH 和上述三行 export 放入用户启动配置；如果 Agent 是由服务或其他启动器运行，则应在对应启动环境中设置，不能只在另一个终端 export。

### 3.4 验证程序和内置模板

```bash
command -v openspec
openspec --version
openspec schemas --json
```

预期：

- `command -v` 指向刚安装的用户目录程序；
- 版本输出 `1.15.0`；
- schemas 中可以看到 `spec-driven`。

内置 schema 和模板随独立程序打包，首次使用会解包到用户数据目录，所以用户配置/数据目录需要可写。**不需要再从源码仓库拷贝模板，也不要求联网拉取模板。**

新终端可重新执行以下四行恢复环境：

```bash
export PATH="$HOME/.local/bin:$PATH"
export OPENSPEC_TELEMETRY=0
export DO_NOT_TRACK=1
export OPENSPEC_NO_UPDATE_CHECK=1
```

## 4. 在项目里初始化：优先选择通用 agents 集成

先在一个独立练习目录熟悉文件布局；本文不会要求你先迁移整个公司项目。

```bash
mkdir -p "$HOME/openspec-demo"
cd "$HOME/openspec-demo"
openspec init --tools agents --language Chinese --no-animation
```

以后接入真实工程时，在已经核对过的工程根目录执行同一个 init 命令。旧 OpenSpec 项目先检查已有文件和版本，不要一上来使用 `--force` 清理旧配置。

为何选 `agents`：

- 它生成通用 `.agents/skills/` 文件；
- 不把内部 CLI 假装成官方 Claude/Codex；
- 即使内部 CLI 不会自动发现技能，你仍可让它显式读取文件。

这只是一个通用文件输出目标，不证明内部 CLI 已原生支持技能自动发现。

本次实际执行输出的关键信息：

```text
Created: Other / Universal (shared .agents skills)
6 skills in .agents/
Commands skipped for: agents (no adapter)
Config: openspec/config.yaml (schema: spec-driven)
```

**`Commands skipped ... no adapter` 在这里是预期结果，不是安装失败。** 因为 agents 集成生成技能，不生成某个厂商的快捷命令。

初始化后主要文件如下。表中路径相对项目根目录：

| 路径 | 用途 |
|---|---|
| `openspec/config.yaml` | 小体积的项目背景、规则和操作建议 |
| `openspec/specs/` | 已整理的系统行为规格 |
| `openspec/changes/` | 当前变更的规划与任务 |
| `.agents/skills/openspec-propose/SKILL.md` | 创建变更和规划产物 |
| `.agents/skills/openspec-explore/SKILL.md` | 只读探索需求或代码 |
| `.agents/skills/openspec-apply-change/SKILL.md` | 根据规划执行任务 |
| `.agents/skills/openspec-update-change/SKILL.md` | 修改现有规划产物 |
| `.agents/skills/openspec-sync-specs/SKILL.md` | 同步变更规格 |
| `.agents/skills/openspec-archive-change/SKILL.md` | 归档流程指引 |

验证生成结果：

```bash
find .agents/skills -name SKILL.md -print
openspec context --json
```

`context` 中的 `root.path` 应是当前练习项目。`init` 只初始化结构和技能；不会自动为你分析业务、写完设计或生成业务代码。

默认 core 集成不包含额外的 verify 工作流，因此本文不要求你必须能输入 `/opsx:verify`。

## 5. 第一次使用：从一个很小的 C 变更开始

教学变更名统一用：`add-null-input-check`。含义为“在配置解析入口增加空输入处理，并验证有效输入行为不变”。真实工程中先确认该行为确实需要修改；已存在的功能不要为了练习重复实现。

流程为：**生成规划 → 人工查看 → 执行一个小任务 → 实际验证 → 归档。**

### 5.1 用内部 Agent 生成规划

打开你平常使用的内部 Agent，工作目录设为已初始化的项目。下面文本既可粘贴到交互对话，也可作为你已跑通的非交互命令的任务输入；本文不猜测内部 CLI 的具体参数。

```text
当前项目已经执行过 OpenSpec 初始化。
请先确认工作目录，然后读取：
.agents/skills/openspec-propose/SKILL.md
按照该文件定义的流程执行 OpenSpec 规划。

本次变更名：add-null-input-check。
目标：在现有 C 工程的配置解析入口处理空输入，沿用已有错误码约定，
保持合法输入行为、公共接口和公共结构体布局不变。

请先检索真实入口、调用关系和相关测试，不要虚构函数名或错误码。
若该行为已经存在或当前目录没有目标 C 工程，请报告事实并停在澄清阶段。
只生成本次变更的 proposal、specs、design 和 tasks，不修改业务代码。
正文写中文，保留 OpenSpec 所需的英文结构标题及 SHALL/MUST。
完成规划后列出生成文件和待确认问题，然后停止。
```

**空练习目录只能练文件流程，不能产生有真实代码依据的设计。** 如果你当前只有空目录，可以改为让 Agent 创建一个标明“教学用”的非敏感 C 示例设计，或把这段提示留到选定练习工程后再使用；不要让它编造公司工程现状。

如果内部 CLI 不支持 `.agents` 自动加载，显式“读取上述文件”就是兼容路径。无需先让 `/opsx:propose` 出现在自动补全里。

通常会生成：

| 变更目录中的文件 | 你需要查看什么 |
|---|---|
| `proposal.md` | 为什么改、改什么、不改什么、影响范围 |
| `specs/<能力名>/spec.md` | 对外行为、边界条件、可验证场景 |
| `design.md` | 实现方案、接口、错误处理、兼容性与风险 |
| `tasks.md` | 可分次完成的小任务及对应验证要求 |

它们位于 `openspec/changes/add-null-input-check/`。如果项目使用其他 schema，文件以实际输出为准。

### 5.2 如果技能方式暂时不好用：用纯 CLI 指令兜底

**此方法与 5.1 二选一。** 如果同名变更已经创建，就不要重复运行 `new change`；直接查看状态。

在 Shell 执行：

```bash
openspec new change add-null-input-check
openspec status --change add-null-input-check --json
openspec instructions proposal --change add-null-input-check --json
```

`new change` 创建变更骨架，**不会自己调用 AI 写完四份文档**。让内部 Agent 读取最后一条命令的输出，按照 `template`、`instruction`、`rules` 和实际输出路径写 proposal。

然后按依赖顺序逐项获取指令，让 Agent 写文件：

```bash
openspec instructions specs --change add-null-input-check --json
openspec instructions design --change add-null-input-check --json
openspec instructions tasks --change add-null-input-check --json
```

每个阶段先完成并查看，再进入下一阶段。若出现 blocked，检查缺的依赖文件，不要用空文件骗过状态判断。路径优先采用指令返回的 `resolvedOutputPath` 或状态中的产物路径，不凭记忆猜目录。

这条路径不依赖任何厂商的快捷命令，只需要内部 Agent 能执行命令、读取结果和写文件。

### 5.3 在 Shell 检查规划

```bash
openspec status --change add-null-input-check --json
openspec validate add-null-input-check --strict --no-interactive
openspec list --json
```

这里必须区分三种状态：

| 结果 | 实际含义 |
|---|---|
| status 的产物 `done` / `isComplete: true` | 相应规划文件存在，不是代码完成 |
| list 的 `completedTasks` / `totalTasks` | 任务清单中勾选了多少项，不是独立测试证明 |
| validate 成功 | OpenSpec 覆盖的规格结构与内容规则通过，不是 C 编译或 LLT 通过 |

本次实测中，四类文件都存在时，status 已显示 `isComplete: true`，但任务进度仍为 **0/3**。不要因此让 Agent直接宣称交付完成。

## 6. 配置适合 C/C++ 项目的规则

初始化的 `openspec/config.yaml` 已包含语言设置。在查看现有内容后合并需要的字段，保留原有项目配置。下面是小规模起步的例子：

```yaml
schema: spec-driven

context: |
  Language: Chinese
  All artifacts must be written in Chinese.
  Keep OpenSpec structural headings and SHALL/MUST keywords in English.
  本项目以 C/C++ 为主；代码与日志只在公司内网处理。
  优先沿用现有模块结构、错误码、内存管理和构建测试流程。

rules:
  specs:
    - 写清正常输入、边界输入和失败场景，要求可以验证。
  design:
    - 明确接口、错误处理、内存所有权和 ABI/结构体布局影响。
    - 区分已从代码确认的事实、假设和待确认事项。
  tasks:
    - 每个任务对应一个可独立检查的小改动。
    - 任务写明相关设计章节、允许修改范围和验证要求。
    - 不把整个模块或整份巨型设计文档合并成一个任务。

operations:
  apply:
    guidance:
      - 按本轮明确指定的任务范围执行，不自动扩大范围。
      - 编译或测试未执行时明确写未验证，不声称通过。
      - 发现需要修改已批准约束时记录问题并停止本次实现。
```

`rules` 的键是产物 ID，例如 `specs`、`design`、`tasks`。**不要把 `apply` 填进 `rules`，它是操作，不是默认 schema 的产物。** 操作建议放在 `operations.apply.guidance`。

这些规则和 guidance 是提供给 Agent 的行为约束，不是权限控制或编译门禁。是否加载、是否遵循仍要检查；真正的构建/测试结果必须由实际工具产生。

## 7. 执行、修改规划和跨会话接续

### 7.1 每次只执行一个明确的小任务

先在 Shell 查看本次实现所需的信息：

```bash
openspec instructions apply --change add-null-input-check --json
```

输出中可以看到 `contextFiles`、任务和完成进度。给内部 Agent 的任务输入示例：

```text
读取 .agents/skills/openspec-apply-change/SKILL.md，
继续变更 add-null-input-check。

先运行 openspec instructions apply --change add-null-input-check --json，
并对照 Git 工作区检查已有修改。
本轮只执行 tasks.md 中“1.1 <替换为实际任务标题>”这一项，完成后停止。
不要顺带执行后续任务。

按要求读取相关规划文件，检索真实代码，执行对应验证。
报告改动文件、执行过的命令、实际结果和仍未验证的内容。
未满足本项验证条件时保留未完成状态。
不要自动归档、提交或推送。
```

把 `<替换为实际任务标题>` 换成真实任务，避免编号发生变化时做错任务。

“每轮只做一项”是这里附加的使用约束。默认 apply 工作流可能连续处理多项待办，因此每轮输入都要明确范围。

### 7.2 实现后仍要运行真实构建和测试

沿用你已有的构建工具、LLT 和检查命令。至少记录：

- 对应的代码版本或工作区修改；
- 实际执行的命令/工具及退出结果；
- 用例数量、失败项或未覆盖项；
- 与本次设计要求的对应关系。

OpenSpec 的勾选和 validate 不能替代这些结果。已有 CI/门禁可以继续使用，不必为了入门先建设新平台。

### 7.3 设计需要修改时

在 AI 中明确请求：

```text
读取 .agents/skills/openspec-update-change/SKILL.md。
修改变更 add-null-input-check 的规划：<填写新增事实或决策>。
先检查 specs、design、tasks 之间的一致性。
本轮只更新规划，不修改业务代码；列出受影响任务和需要重新验证的内容。
```

注意名称容易混淆：

| 操作 | 用途 |
|---|---|
| 让 AI 执行 `openspec-update-change` 技能 | 修改当前需求/设计/任务规划 |
| 在 Shell 执行 `openspec update` | 根据已安装的 CLI 刷新生成的技能/命令文件 |
| 替换 Release 二进制 | 升级 OpenSpec 程序本身 |

### 7.4 新会话继续任务

```text
继续 OpenSpec 变更 add-null-input-check。
先读取 openspec/config.yaml 和
.agents/skills/openspec-apply-change/SKILL.md。
运行 status、list 和 instructions apply，核对当前任务与 Git 工作区。
不要直接信任旧聊天摘要，不要覆盖尚未验收的已有修改。
报告已确认的进度、未验证修改和下一项任务，再执行我本轮指定的范围。
```

OpenSpec 能重新提供文档和任务状态，但不会自动为你的内部 CLI 恢复模型会话、创建进程检查点或核对所有测试证据。

## 8. 大设计文档如何接入，避免再次塞满 200K

第一轮只把**当前一个小变更**接入 OpenSpec，不把整个模块的历史设计、所有需求和全量验证日志一次迁入。

建议：

1. `context` 只写短小的项目背景和硬约束，不粘贴巨型设计全文。
2. 按真实行为能力组织 spec，而不是一个 spec 包含整个模块。
3. 每个 change 限定一组紧密相关的行为修改，保持 proposal/design/tasks 可读。
4. 既有大设计保留原文，任务记录准确章节、需求 ID、公共约束和源文档版本。
5. 日志保留完整文件，反馈到会话的是结果、失败摘要和精确位置；需要时再读原文。

**当前默认 apply 工作流要求读取 `contextFiles` 中的所有文件。** 因此不能声称 OpenSpec 已自动做到“面对任意大文档，只读取当前相关章节”。如果 change 自身的设计和规格仍非常大，应先拆变更或另行设计章节读取机制，不要只把文件改名后继续全量读取。

重要接口、状态机、内存所有权和布局约束要能回溯原文。摘要用于导航，不能作为唯一事实来源。这一节是使用建议，不是已验证的自动大文档检索功能。

## 9. 完成后归档

先确认：规划已经审核、代码验证有真实结果、任务状态与实际工作一致。然后在 Shell 执行：

```bash
openspec validate add-null-input-check --strict --no-interactive
openspec archive add-null-input-check
```

首次使用保留交互提示，查看它要更新哪些规格，不主动加跳过验证的参数。

对默认 schema，CLI 归档会把 delta 合并到 `openspec/specs/`，并将变更目录移到：

```text
openspec/changes/archive/YYYY-MM-DD-add-null-input-check/
```

归档后检查：

```bash
openspec list --json
openspec validate --all --strict --no-interactive
```

归档不是 Git 提交，也不是代码验收。后续提交按你原来的研发流程进行；不要手工移动目录来冒充 OpenSpec 归档。

## 10. 常见问题

| 现象 | 处理 |
|---|---|
| 下载仓库后不知道怎么运行 | 安装用 Release 的 `openspec-linux-*`；源码 ZIP 只用于查阅，不需要构建 |
| `Permission denied` | 检查可执行权限、目录访问权限与挂载的 `noexec`；已使用 install 0755 时重点检查挂载策略 |
| `Exec format error` | 检查 x86_64 / arm64 是否下载错；不要尝试通过 chmod 修复架构不匹配 |
| 明明文件存在却提示找不到程序/加载器 | 可能是 glibc/musl 版本选错、动态加载器缺失，或下载成 HTML 页面；用 `file`、`ldd`（若系统具备）辅助确认 |
| `GLIBC_x.xx not found` | 目标系统运行库太旧；不能仅靠下载源码解决，也不要盲目替换系统 libc；需匹配运行环境 |
| musl 程序缺 `libstdc++` / `libgcc` | 目标系统缺基础运行库；本文不假设能联网安装，需要在内网另行预置或换匹配环境 |
| `openspec: command not found` | 重新 export PATH，或用 `$HOME/.local/bin/openspec --version` 直接验证 |
| 版本显示不是 1.15.0 | 用 `command -v openspec` / `type -a openspec` 找旧程序，核对 PATH，不继续混用 |
| 初始化显示 `Commands skipped ... no adapter` | agents 目标只生成技能，属预期行为 |
| 内部 AI 不认识 `/opsx:propose` | 显式让它读取 `.agents/skills/openspec-propose/SKILL.md`，不要假定厂商快捷命令兼容 |
| 输入 `openspec propose` 或 `openspec apply` 不工作 | 规划/实现由 Agent 技能执行；Shell 用的是 `new change` 和 `instructions ...` 等 CLI 命令 |
| `new change` 后只有少量骨架文件 | 正常；CLI 未替你调用 AI，文档由 Agent 按 instructions 生成 |
| status 完成，但代码没改 | status 首先反映产物存在性，另看任务进度和真实验证结果 |
| strict 校验失败 | 先看诊断，检查能力路径、Requirement/Scenario 结构、SHALL/MUST、场景正文等，不跳过校验 |
| 每轮还是读很多文档 | 检查当前 change 的规模和 contextFiles；OpenSpec 不会自动把任意大设计精准裁切 |
| 换会话规则又没加载 | 使用第 7.4 节接续提示；文件存在不代表宿主已经自动注入 |
| 用户数据目录无法写入 | 首次解包模板需要可写目录；确认默认用户目录或按公司规范设置 XDG_CONFIG_HOME/XDG_DATA_HOME |

## 11. 后续更新仍走手动 Release 路线

这份入门文档锁定 v1.15.0。以后需要升级时：

1. 在能下载 GitHub Release 的环境获取新版本程序和同版本 SHA256SUMS；
2. 在目标机校验后替换用户目录中的二进制，并保留可回退的旧版本；
3. 核对 `openspec --version`；
4. 在项目里先执行 `openspec update --dry-run --json` 查看将变动的生成文件，再决定是否执行 `openspec update`；
5. 查看项目变更，确认规格、设计和任务没有被意外改动。

`openspec update` 的本意是刷新项目集成；它不等于从 GitHub 手动替换二进制。关闭版本检查的环境中，也不依赖它发现新版本。

## 12. 本文做过什么验证

本次实际下载并执行官方 `openspec-linux-x64`，未构建源码。校验和与同版本 SHA256SUMS 一致。环境为 Linux x86_64、glibc 2.39。

已在隔离的练习目录和独立配置/数据目录中实际执行：

- 版本、帮助、内置 schema 和模板查询；
- `init --tools agents --language Chinese --no-animation`，生成六个通用技能；
- 变更骨架创建、context/status/list、规划产物指令读取；
- 使用人工准备的教学文档夹具进行 strict 规格校验；
- 实现指令输出、归档合并和归档后规格校验。

共 19 次上述 CLI 调用返回成功。夹具里的完成勾选只用于验证 CLI 状态和归档行为，不代表真实 C 代码编译、LLT 或 AI 编码成功。

未验证：你的内部 CLI 技能自动发现和模型执行效果、ARM64/musl 实机兼容、你的目标机运行库、完整断网/外联审计、生产工程集成。已核对遥测和更新关闭开关，并在设置这些变量后执行上述流程；不把这一点夸大为整套 Agent 已通过无外联审计。

## 官方依据

- [v1.15.0 Release：独立二进制和 SHA256SUMS](https://github.com/Fission-AI/OpenSpec/releases/tag/v1.15.0)
- [Release 附件列表](https://github.com/Fission-AI/OpenSpec/releases/expanded_assets/v1.15.0)
- [安装说明](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/docs/installation.md)
- [Linux 安装器：平台识别、校验和手动运行检查](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/scripts/binary/install.sh)
- [独立程序构建源码：内置 schema 和平台列表](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/scripts/binary/build.mjs)
- [官方独立程序冒烟检查](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/scripts/binary/smoke.sh)
- [终端命令与 AI 工作流的区别](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/docs/how-commands-work.md)
- [支持的工具及通用 agents 集成](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/docs/supported-tools.md)
- [CLI 命令参考](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/docs/cli.md)
- [项目规则字段](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/src/core/project-config.ts)
- [遥测关闭信号](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/src/telemetry/opt-out.ts)
- [更新检查开关](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/src/core/version-check.ts)
- [apply 的默认上下文文件读取流程](https://github.com/Fission-AI/OpenSpec/blob/v1.15.0/src/core/templates/workflows/apply-change.ts)

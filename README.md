# PTDD · 从需求到 TDD 实施的轻量开发工作流

面向互联网公司的产品经理（PM）、全栈工程师及前后端开发人员的中文 Agent Skill。适用于 vibe coding 中的新功能、功能增强、重构和 Bug 修复：先对齐项目与需求，再定义行为契约和实施计划，最后按 TDD 验证交付。

PTDD 保留规范驱动开发（SDD）的契约先行思路与 RED → GREEN → REFACTOR 流程，按需读取规则，不要求先建立一整套初始化档案。它是一套供 Codex 执行的工作指引，不是编译器、测试框架或自动验收系统。

## 成果展示

**案例：用 TDD 为 Python 分页参数补齐类型、边界与上限行为。**

合成项目最初只有 `normalize_limit(value): return value` 和一项正常输入测试。需求是拒绝布尔值、非整数及非正数，并把大于 100 的值截断为 100。

2026-10-15 在 macOS、Codex CLI 0.162.1、`gpt-6.1-sol`、Python 3.14.7、coverage 7.16.2 上，从本仓库下载并隔离安装后完成阶段 0–4。生成了行为契约、TDD 计划、正式测试和最小实现；纯库函数的阶段 5 标为不适用。此次案例只证明该 Python 路径，不扩展为其他语言均已实测。

### 实际交付与验证

- 基线：1 项已有测试通过。
- RED：7 项测试被发现，16 个子场景因未抛异常或未截断而失败，退出码 1。
- GREEN：实现后同一组 7 项测试通过，退出码 0。
- REFACTOR 检查：当前最小实现无需额外重构，覆盖率运行复跑全部测试仍通过。

下面是实际覆盖率输出摘录，统计全部 `pagination.py`，未增加排除项：

```text
Name            Stmts   Miss Branch BrPart  Cover   Missing
-----------------------------------------------------------
pagination.py       8      0      6      0   100%
-----------------------------------------------------------
TOTAL               8      0      6      0   100%
```

实际生成的最小实现：

```python
def normalize_limit(value):
    if isinstance(value, bool) or not isinstance(value, int):
        raise TypeError('limit must be an integer')
    if value <= 0:
        raise ValueError('limit must be positive')
    if value > 100:
        return 100
    return value
```

未验证：独立类型检查、构建和 lint（合成项目未配置）。没有安装无关依赖、提交或推送代码；这不是通用可靠性或性能基准。

<details>
<summary>查看实际生成的行为契约</summary>

# normalize_limit 行为契约

创建日期：2026-10-15。来源：用户明确需求及 pagination.py。
关联计划：agent_docs/plans/2026-10-15-normalize-limit-plan.md。

## 输入输出与错误

函数 normalize_limit(value) 先检查类型，接受 int（含子类），明确拒绝 bool。
非 int 或 bool 抛 TypeError；合法整数小于等于 0 抛 ValueError。
1–100 原样返回，大于 100 返回整数 100。无副作用，不做类型转换。
异常消息不属于本次契约。

## 场景与断言

| 场景 | 输入 | 预期及断言 |
| --- | --- | --- |
| 正常及边界 | 1、20、99、100 | 返回原值，类型为 int |
| 截断 | 101、10**100 | 返回整数 100 |
| 非正数 | -10、-1、0 | 抛 ValueError |
| 布尔值 | True、False | 抛 TypeError，优先于数值检查 |
| 非整数 | None、字符串、浮点数、列表、字典、对象 | 抛 TypeError |
| 整数子类 | int 子类的正整数 | 原样返回 |

## 接口测试包适用性

本次无外部 API 变更。纯库函数不涉及 HTTP，阶段 5 和 Apifox 不适用。

</details>

<details>
<summary>准备合成项目并复制请求，复现完整流程</summary>

先按下方说明安装 `$ptdd`，并确认 Python 3.10+ 与已登录的 Codex 可用。在一个可写目录运行以下 shell 命令（新建 `ptdd-demo`，同名目录已存在时请另选位置；coverage 安装需要网络）：

```sh
(
  set -eu
mkdir ptdd-demo
cd ptdd-demo
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install 'coverage==7.16.2'
cat > pagination.py <<'PYCODE'
def normalize_limit(value):
    return value
PYCODE
mkdir tests
cat > tests/test_pagination.py <<'PYCODE'
import unittest
from pagination import normalize_limit

class ExistingBehavior(unittest.TestCase):
    def test_valid_limit(self):
        self.assertEqual(normalize_limit(20), 20)
PYCODE
cat > README.md <<'DOC'
合成 Python 分页库，无 HTTP、数据库或网络依赖。
测试：python3 -m unittest discover -s tests -v
覆盖率：python3 -m coverage run --branch --source=pagination -m unittest discover -s tests -v
报告：python3 -m coverage report -m --fail-under=80
生产代码统计范围：pagination.py，不排除任何生产行；行覆盖率至少 80%，同时报告分支覆盖。
不提交 Git。
DOC
)
```

然后运行 `codex -C ptdd-demo`，或用桌面端打开 `ptdd-demo`，复制以下对话请求：

```text
$ptdd 为当前合成 Python 项目增强 normalize_limit(value)。请按六阶段思路完成阶段 0–4，阶段 5 因非 HTTP 不适用。契约已明确：只接受 int 且拒绝 bool，非 int 抛 TypeError；小于等于 0 抛 ValueError；1–100 原样返回，大于 100 截断为 100。
本次明确授权保存本地契约和计划并开始实施，范围限定 pagination.py、tests/test_pagination.py 及 agent_docs 中对应文档；不需要逐阶段重复确认。保留现有技术栈，先测试确认有效 RED，再最小实现 GREEN，必要重构并复跑。使用项目 README 的 unittest 和覆盖率入口；不要安装其他依赖，不提交或推送 Git。结束时列实际证据、已通过和未验证项。
```

预期会修改这个练习项目的生产文件、正式测试和 `agent_docs` 文档；不触碰真实业务项目，不上传。测试数量、措辞可以变化，但须有有效 RED、对应 GREEN、契约边界和真实覆盖率证据。查看结果后，可自行删除整个练习目录；不要把删除命令用于其他项目。

</details>

## 适用人群与场景

- **PM / 技术产品经理**：把模糊需求梳理为目标、边界、可观察行为与验收条件，和开发共同确认契约及计划。
- **全栈与前后端工程师**：让 AI 基于现有架构和工具链实施需求，减少套用错误技术栈、先写代码再补测试、遗漏边界等问题。
- **使用 vibe coding 的个人或小团队**：希望保持短反馈，同时保留需求确认、行为约定、测试证据和交付边界。

PM 可以使用澄清、契约和计划环节；真正实施与验收仍需要可用的开发环境。纯文案修改或一次性咨询通常无需完整开发流程。

## 六阶段流程

| 阶段 | 内容 | 主要结果 |
| --- | --- | --- |
| 0 · 上下文与偏好加载 | 识别目标项目/模块，按需读取项目指令、配置、锁文件、CI 和同类实现 | 技术栈、规范、验证命令及待确认条件 |
| 1 · 需求澄清 | 一次一个关键问题，比较方案与权衡，遵循 YAGNI | 对齐目标、范围与设计方向 |
| 2 · 接口契约 | 先约定输入输出、错误、边界、副作用和断言 | 轻量行为契约；不限于 HTTP |
| 3 · 实施计划 | 明确文件、结构、TDD 任务、命令和验收标准 | 可执行计划，引用契约而不重复 |
| 4 · TDD 实施 | 有效 RED → 最小实现 GREEN → REFACTOR，执行约定验收 | 代码、正式测试及实际验证结果 |
| 5 · 接口测试包（可选） | 为适用的 HTTP 行为生成 Postman Collection，供 Apifox/Postman 导入 | 请求、curl、变量与行为断言 |

阶段 0→1、1→2 自然衔接；契约与计划先展示、确认后保存。阶段 2、3、4 完成后提示下一阶段并等待确认；只有明确授权实施才进入阶段 4。阶段 5 可跳过，非 HTTP 需求不强造接口测试包。未经授权不提交或推送代码。

偏好（怎么写）、契约（表现如何）、计划（怎么改）分开维护。默认产物路径：

```text
agent_docs/contracts/YYYY-MM-DD-<feature>-contract.md
agent_docs/plans/YYYY-MM-DD-<feature>-plan.md
agent_docs/api-tests/YYYY-MM-DD-<feature>.postman_collection.json
```

具体保存位置仍遵守目标项目政策和用户确认。

## 跨语言与跨项目

不预设 Go、TypeScript、Python 或某个框架。阶段 0 根据目标模块的实际证据选择工具、目录和验证入口；多语言仓库分别识别相关模块。识别本身只读，不安装依赖，不运行构建或测试。

不要求 AGENTS.md、L2 规则或初始化配置存在；已有材料足够就直接复用。切换项目时重新绑定上下文，只有实际规则冲突或验证缺口才简短提醒，不根据语言变化推断能力下降，也不自动派生新 skill。

默认附带的 [L1 偏好](preferences/global-preferences.md) 包含中文注释、文档及人类可读输出、复用现有组件、不使用 emoji、覆盖率至少 80% 等约定。覆盖率须明确实际指标与生产代码范围，项目更严格要求继续生效。请先了解这些偏好；用户要求与宿主生效的项目指令优先，适用范围内再解析 L2 > L1 > 项目证据 > 通用默认。少量确需保存的项目特殊规则可由用户要求写入 `agent_docs/preferences/project-rules.md`，并非安装前置条件。

## 安装到 Codex

需要已安装并登录的 Codex；以下命令适用于 macOS/Linux 的 sh/bash/zsh。先用 `codex --version` 确认 CLI 可用；仅用桌面端时也可执行文件安装步骤，再在桌面端打开目标项目。

### 1. 获取发布目录

公开仓库：[score2021/ptdd](https://github.com/score2021/ptdd)。使用 Git 下载：

```sh
git clone https://github.com/score2021/ptdd.git
cd ptdd
```

本地目录或从 GitHub 下载 ZIP 并解压也可以；先进入包含 `SKILL.md`、`rules/`、`preferences/` 和 `LICENSE` 的发布目录，再执行下一步。

### 2. 用户级安装（所有项目共用）

复制整个 skill 的必要文件到 Codex 的用户级发现目录。整块执行；如果目标已存在（包括软链接），命令会停止，不覆盖旧版本：

```sh
(
  set -eu
  test -f SKILL.md
  test -d rules
  test -d preferences
  test -f LICENSE
  ptdd_dest="$HOME/.agents/skills/ptdd"
  if [ -e "$ptdd_dest" ] || [ -L "$ptdd_dest" ]; then
    printf '%s\n' "已存在 $ptdd_dest，请先检查现有版本；本次未覆盖。" >&2
    exit 1
  fi
  mkdir -p "$ptdd_dest"
  cp SKILL.md LICENSE "$ptdd_dest/"
  cp -R rules preferences "$ptdd_dest/"
  printf '%s\n' "已安装到 $ptdd_dest"
)
```

`~/.agents/skills/ptdd/` 是此方法的实际安装与发现地址。Codex 支持软链接；部分旧安装可能在 `~/.codex/skills/ptdd/` 保存实际文件并从这里链接。不要额外重复安装同名版本。更新前检查旧版本路径、个人改动和备份，再替换必要文件；不要直接覆盖自定义偏好。

Codex 会检测 skill 变化，下一轮即可尝试使用；若列表未刷新，重启 Codex。CLI 可运行 `/skills` 或输入 `$` 查找 `ptdd`。路径、软链接和调用方式参见 [OpenAI 官方 Skills 文档](https://learn.chatgpt.com/docs/build-skills)。

### 3. 在目标项目调用

CLI：

```sh
cd /path/to/your-project
codex
```

进入 Codex 后输入（这是对话内容，不是终端 shell 命令）：

```text
$ptdd 为当前项目增加列表筛选功能。先识别项目并澄清需求，暂不实施。
```

在 Codex 桌面端打开目标项目，在对话中输入相同内容。需要多语言仓库中的特定模块时，直接指定：

```text
$ptdd 本次只处理 apps/web，先识别该模块，再分析表单校验需求。
```

确认契约和计划后，明确回复“开始实施，按已确认计划执行 TDD”。每个项目建议独立对话，无需重复安装或初始化；新对话要提供任务目标，继续旧任务时指出已有契约和计划。

仅检查项目识别、不进入后续阶段：

```text
$ptdd 仅执行阶段 0，报告当前项目的技术栈、规范和验证入口，暂不修改文件或运行测试。
```

CLI 也可直接进行只读调用（单引号保留 `$ptdd` 字面量）：

```sh
codex exec --sandbox read-only '$ptdd 仅执行阶段 0，识别当前项目并报告验证入口，不修改文件、不运行测试；完成后停止。'
```

需在受信任的目标项目目录中执行，并已完成 Codex 登录。识别输出应区分“配置声明”和“实际验证”；不能因为看到测试命令，就声称测试已通过。

独立安装此目录不会注册 `/ptdd` 或 `/apitest` 自定义斜杠命令。需要为已有 HTTP 接口独立生成测试包时，可在对话中明确请求使用 `$ptdd`，仅为指定接口执行测试包流程。

## 目录

```text
ptdd/
├── README.md
├── LICENSE
├── .gitignore
├── SKILL.md
├── preferences/
│   ├── global-preferences.md
│   └── project-rules.template.md
└── rules/
    ├── project-context.md
    ├── preferences.md
    ├── brainstorm.md
    ├── contract.md
    ├── plan.md
    ├── plan-conventions.md
    ├── plan-field-mapping.md
    ├── plan-git-hygiene.md
    ├── tdd.md
    └── api-test-pack.md
```

本目录仅分发 PTDD，不包含其他 skill、项目代码、凭据、会话记录或临时验证资产。

## 验证范围与限制

此版本在本地隔离的 Go、TypeScript、Python 小型示例中完成过实际 Codex 调用，观察到测试先因目标行为缺失而失败、最小实现后通过，以及相应测试/构建/类型检查（按项目适用项）成功；连续切换项目的阶段 0 识别与安装后的按名称加载也已验证。它们是有限示例，不代表所有语言、框架和业务项目均已验证。

运行 TDD 仍需要项目实际工具链、依赖和必要服务。测试结果、覆盖率、构建与类型检查分别报告；缺环境或零测试不算通过。生成 Collection 的静态检查不等于真实接口调用或所有 Apifox/Postman 版本导入成功。

## 许可证

[MIT](LICENSE)。

# 居委会大妈审 TS 🏠

> 技术是真的，嘴是碎的。一个带居委会大妈人格的 TypeScript Code Review Skill。

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-blue.svg)](https://www.typescriptlang.org/)
[![Skill](https://img.shields.io/badge/Agent-Skill-purple.svg)](https://qwenwork.cn/docs)
[![Stars](https://img.shields.io/github/stars/jianglinguan9-dot/居委会大妈审TS.svg)](https://github.com/jianglinguan9-dot/居委会大妈审TS)

---

## 这是什么

**居委会大妈审 TS** 是一个 Agent Skill——把 AI 变成"王姐"，居委会技术调解员，用唠嗑的口吻帮你做 TypeScript Code Review。

**功能是真的在干活**：六个维度逐项扫描你的 TS 代码，给出准确的问题定位和可操作的修复代码。**但说话方式让人忍俊不禁**：bug 叫"疑点"，`any` 是"法外狂徒"，函数套五层是"套娃"，评分用"社区和谐分"。

### 大妈审代码是什么体验

> 小猿啊，你这 `any` 大妈看得直摇头。`any` 这玩意儿就跟你说"随便吃随便喝"一样，出了事谁都说不清。

> 小猿，你这函数叫 `CalculateTotal`，首字母大写了。TS 里函数名跟变量名一样，都用小驼峰。大驼峰是给类和接口留的，你这是抢了别人的帽子戴。

> 小猿啊，你把 `API_URL` 从 `https` 改成了 `http`？这跟你把家门从防盗门换成纸糊的一样。

## 六大审查维度

| 维度 | 大妈查什么 |
|---|---|
| 类型安全 | `any` 滥用、`as` 断言无校验、缺失返回类型、`!` 非空断言 |
| 命名规范 | 驼峰/蛇形混用、Boolean 前缀、常量风格、`I` 前缀 |
| 结构复杂度 | 嵌套深度、函数行数、文件行数、圈复杂度 |
| 边界与安全 | null/undefined 未处理、异常静默吞掉、Promise 未 catch |
| 死代码检测 | 废弃函数、无用 import、不可达代码、注释代码块 |
| TS 专属习俗 | type vs interface、enum 坑、readonly、泛型约束、satisfies、import type |

## 社区评分系统

每次审完给一个"社区和谐分"，满分 100。

| 扣分 | 级别 | 大妈说法 |
|---|---|---|
| -30 | 致命 | 这可不行，你妈要是知道得念叨你 |
| -15 | 警告 | 这事儿大妈得说说你了 |
| -5 | 建议 | 顺嘴提一句啊 |

| 分数 | 评级 | 大妈评语 |
|---|---|---|
| 90-100 | 模范住户 | 回去给大妈带个橘子 |
| 70-89 | 需要整改 | 还行，有几处得收拾收拾 |
| 50-69 | 限期整改 | 限期改完，大妈回头还来查 |
| 0-49 | 限期搬离 | 这代码属于危房，赶紧重建 |

## 快速开始

### 安装

**方式一：克隆到 Skills 目录（原生支持 SKILL.md 的工具）**

```bash
# QwenWork / 千问办公
git clone https://github.com/jianglinguan9-dot/ts-auntie-review.git ~/.qwenworkcn/skills/ts-auntie-review

# Claude Code
git clone https://github.com/jianglinguan9-dot/ts-auntie-review.git ~/.claude/skills/ts-auntie-review

# WorkBuddy
git clone https://github.com/jianglinguan9-dot/ts-auntie-review.git ~/.workbuddy/skills/ts-auntie-review
```

**方式二：作为系统提示词 / 自定义规则使用**

对于不支持 SKILL.md 目录结构的工具，将 `SKILL.md` 的内容复制到对应配置文件中：

| 工具 | 配置方式 |
|---|---|
| Cursor | 将 SKILL.md 内容粘贴到 `.cursor/rules/ts-auntie-review.mdc` |
| Windsurf | 将 SKILL.md 内容粘贴到 `.windsurfrules` 或项目级 rules |
| Cline | 将 SKILL.md 内容粘贴到 `.clinerules/ts-auntie-review.md` |
| Codex / ChatGPT | 将 SKILL.md 内容作为 system prompt 或 custom instructions |
| GitHub Copilot Chat | 将 SKILL.md 内容粘贴到 `.github/copilot-instructions.md` |
| Aider | 启动时加 `--read SKILL.md` 参数 |

**方式三：一键安装（通用脚本）**

```bash
# 下载到任意位置后，按你的工具引用
git clone https://github.com/jianglinguan9-dot/ts-auntie-review.git ~/ts-auntie-review
```

然后将 `SKILL.md` 的路径或内容按你的工具要求引用即可。

### 使用

安装后，在任何 AI 编程助手中粘贴 TypeScript 代码，说：

> 审一下代码

或：

> review my TS

大妈就上场了。

### 示例

**输入：**

```typescript
function getUserData(id: string): any {
  const res = await fetch(`/api/user/${id}`)
  return res.json()
}
```

**大妈输出（节选）：**

> 小猿啊，来了？大妈瞅一眼啊。
>
> **这可不行，你妈要是知道得念叨你** —— `getUserData` 返回值用了 `any`
>
> 小猿，你这个 `any` 大妈看得直摇头。`any` 这玩意儿就跟你说"随便吃随便喝"一样，出了事谁都说不清。
>
> **这可不行，你妈要是知道得念叨你** —— `getUserData` 用了 `await` 但没标 `async`
>
> 小猿，你这函数里头用了 `await fetch(...)`，但函数压根没标 `async`！这跟你说"我去取个快递"但连门都没出一样。
>
> **社区和谐分：40/100 —— 限期搬离**

完整示例见 [examples.md](examples.md)。

## 文件结构

```
ts-auntie-review/
├── SKILL.md        # 主指令（人格设定 + 审查维度 + 评分系统 + 输出格式）
├── reference.md    # TS 详细规则（satisfies、enum、import type、泛型约束等）
├── examples.md     # 7 个完整审查示例
├── LICENSE         # MIT
└── .gitignore
```

## 大妈的诚实原则

1. **不确定要明说** —— 遇到拿不准的情况，明确指出哪里不确定、为什么，不硬给建议装懂。
2. **接受纠错** —— 你说"这个不是问题"，大妈大方承认，不犟嘴。
3. **结尾免责提醒** —— "大妈的建议你参考着来，有拿不准的自己再查查，大妈也有一把年纪了，难免看走眼。"

## 支持的 AI 工具

**原生支持 SKILL.md 目录结构：**

- [千问办公 / QwenWork](https://qwenwork.cn) — 放到 `~/.qwenworkcn/skills/` 即自动加载
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — 放到 `~/.claude/skills/` 即自动加载
- WorkBuddy — 放到 `~/.workbuddy/skills/` 即自动加载

**通过自定义规则 / 系统提示词适配：**

- [Cursor](https://cursor.com) — `.cursor/rules/ts-auntie-review.mdc`
- [Windsurf](https://codeium.com/windsurf) — `.windsurfrules`
- [Cline](https://cline.bot) — `.clinerules/ts-auntie-review.md`
- [OpenAI Codex / ChatGPT](https://openai.com) — 作为 system prompt 或 custom instructions
- [GitHub Copilot Chat](https://docs.github.com/copilot) — `.github/copilot-instructions.md`
- [Aider](https://aider.chat) — `aider --read SKILL.md`
- 任何支持 system prompt 的 AI 编程工具

## 贡献

欢迎提 Issue 和 PR！特别欢迎：
- 新的审查示例（大妈审更多类型的代码）
- reference.md 规则补充
- 翻译（英文版大妈？）
- 其他语言的"大妈"——Python 大妈、Rust 大妈、Go 大妈

## License

[MIT](LICENSE) — 随便用，出了事别找大妈。

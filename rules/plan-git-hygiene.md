---
title: Planning 细则 — Git Hygiene 与提交范围
impact: HIGH
tags: planning, git, gitignore
trigger: 每次生成实施计划
---

# Git Hygiene 与提交范围

## 1. 计划至少写清

- 预期提交的功能代码、正式回归测试、必要配置/迁移与契约资产。
- 临时实验、日志、覆盖率/构建输出、缓存和会话文件。
- 是否确需补充忽略规则，以及依据的项目政策。

## 2. 区分源码与临时产物

正式测试源码与必要 fixture/golden 默认属于可复现交付，不得因为使用 AI 或 TDD 就忽略。不要添加 `*_test.go`、`*.test.ts`、`test_*.py`、整个测试目录等宽泛忽略规则。

按实际工具忽略临时输出，不复制固定语言清单。AGENTS.md、agent_docs、契约与测试包是否提交遵循项目政策，不能全局默认忽略；必要团队规范应可共享。仅本机使用的忽略需求可采用 `.git/info/exclude`，不无故修改共享规则。

## 3. 收尾核对

有 Git 时用 `git status --short`、必要时 `git diff --stat` / `git check-ignore -v <path>` 核对范围；保护用户已有改动，不清理未知文件，不用强制暂存绕过误配置。非 Git 项目检查本次改动清单，不为卫生检查自动初始化仓库。

未经用户授权，不执行 git add、commit 或 push。

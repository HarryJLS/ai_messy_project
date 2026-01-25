# AI Messy Project

Claude Code Skills 和 Workflow 工具集，提供结构化的 TDD 开发工作流、多语言单元测试支持和 UI/UX 设计智能。

## 项目概览

本项目是一个 **Claude Code 增强工具集**，包含：

- **Agent 工作流**：基于 TDD 的结构化开发流程，支持任务管理、日志隔离、上下文恢复
- **代码质量工具**：Code Review、Code Fixer、Code Simplifier
- **单元测试工具**：自动检测项目语言，生成符合最佳实践的单元测试
- **UI/UX 设计智能**：50+ 风格、97+ 配色方案、57+ 字体搭配
- **Skill 管理**：创建、打包、同步 Claude Code Skills
- **WorkTeam 工作流**：5 人角色分工的产品开发流水线

## 目录结构

```
ai_messy_project/
├── skills/                 # Claude Code Skills (可安装的技能)
├── workflow/               # Workflow 文档 (独立使用的工作流)
├── common/                 # 共享参考文档
├── CLAUDE.md               # Claude Code 项目指南
├── .claudeignore           # Claude Code 忽略配置
└── .gitignore              # Git 忽略配置
```

---

## 📁 skills/ - Claude Code Skills

可安装到 Claude Code 的技能模块，通过 `/skill-name` 命令调用。

### Skills 总览

| 分类 | Skill | 命令 | 用途 |
|------|-------|------|------|
| **Agent 工作流** | plan-init | `/plan-init` | 初始化项目，创建 `features.json` 和 `logs/` |
| | plan-next | `/plan-next` | 执行下一个任务 (TDD: RED → GREEN → COMMIT) |
| | plan-log | `/plan-log` | 手动记录架构决策、紧急修复等 |
| | plan-archive | `/plan-archive` | 归档已完成工作 |
| **代码质量** | code-review | `/code-review` | 审查代码变更，生成审查报告 |
| | code-fixer | `/code-fixer` | 自动修复代码规范问题 |
| | code-simplifier | `/code-simplifier` | 简化优化代码，提升可维护性 |
| **测试** | unit-test | `/unit-test` | 自动检测语言，生成单元测试 |
| **设计** | ui-ux-pro-max | `/ui-ux-pro-max` | UI/UX 设计智能助手 |
| **Skill 管理** | skill-creator | `/skill-creator` | 创建和打包 Claude Code Skill |
| | add_or_update_skill | `/add_or_update_skill` | 同步 skill 到多个平台 |

### 目录结构

```
skills/
├── plan-init/              # 初始化 Agent 框架
│   └── SKILL.md
├── plan-next/              # 执行下一个待处理任务 (TDD 循环)
│   └── SKILL.md
├── plan-log/               # 手动记录非任务进度
│   └── SKILL.md
├── plan-archive/           # 归档已完成的工作
│   └── SKILL.md
├── code-review/            # 代码审查
│   ├── SKILL.md
│   ├── java.md             # 阿里巴巴 Java 规范检查清单
│   ├── go.md               # 字节跳动 Go 规范检查清单
│   ├── frontend.md         # React/TypeScript 检查清单
│   └── backend.md          # Python/FastAPI 检查清单
├── code-fixer/             # 代码自动修复
│   ├── SKILL.md
│   └── references/
│       ├── java.md         # Java 修复规则
│       ├── go.md           # Go 修复规则
│       ├── frontend.md     # Frontend 修复规则
│       └── backend.md      # Backend 修复规则
├── code-simplifier/        # 代码简化优化
│   └── SKILL.md
├── unit-test/              # 单元测试生成
│   ├── SKILL.md
│   └── references/
│       ├── go-mockey-testify.md    # Go 测试指南
│       ├── java-spock.md           # Spock 测试指南
│       ├── java-junit.md           # JUnit 5 测试指南
│       └── java-dependencies.md    # Java 依赖配置
├── ui-ux-pro-max/          # UI/UX 设计智能
│   ├── SKILL.md
│   └── scripts/            # 设计系统搜索工具
├── skill-creator/          # Skill 创建指南
│   ├── SKILL.md
│   └── references/
│       ├── workflows.md    # 工作流设计模式
│       └── output-patterns.md  # 输出模式
├── add_or_update_skill/    # Skill 管理工具
│   └── SKILL.md
└── *.skill                 # 打包后的技能文件 (ZIP 格式)
```

---

## Skill 详细说明

### 1. Agent 工作流 (plan-*)

基于 TDD 的结构化开发工作流，核心特性：

- **抗遗忘**：通过任务日志恢复上下文
- **抗范围蔓延**：JSON 定义范围，日志提供详情
- **任务隔离**：每个任务独立日志文件 (`logs/task-{id}.log`)
- **TDD 强制**：必须先 RED 再 GREEN

**执行流程：**
```
READ → EXPLORE → PLAN → RED 🔴 → IMPLEMENT → GREEN 🟢 → COMMIT
```

| 命令 | 触发词 | 说明 |
|------|--------|------|
| `/plan-init` | "初始化项目"、"开始新项目"、"创建任务列表" | 创建 `features.json` 和 `logs/` 目录，交互式定义任务 |
| `/plan-next` | "执行下一个任务"、"继续任务"、"开始开发" | 执行七阶段 TDD 循环 |
| `/plan-log` | "记录进度"、"写日志"、"记录决策" | 手动记录非任务进度 |
| `/plan-archive` | "归档项目"、"清理工作区"、"备份任务" | 归档到 `archives/YYYY-MM-DD-HHMMSS/` |

**核心文件：**
- `features.json` - 任务的单一事实来源
- `logs/init.log` - 初始化日志
- `logs/task-{id}.log` - 每个任务独立日志

---

### 2. Code Review

基于 diff 的代码审查，自动检测语言并应用对应规范。

| 触发词 | 说明 |
|--------|------|
| `/code-review` | 审查代码变更 |

**自动检测域：**

| 文件模式 | 域 | 规范 |
|----------|-----|------|
| `*.java` | Java | 阿里巴巴 Java 开发规范 |
| `*.go` | Go | 字节跳动 Go 开发规范 |
| `*.tsx`, `*.jsx`, `designer/` | Frontend | React/TypeScript 最佳实践 |
| `*.py`, `server/`, `rag/` | Backend | Python/FastAPI 最佳实践 |

**审查类别：**
- Security (Critical): 硬编码密钥、命令注入、eval 执行
- Code Quality: console.log、TODO 注释、空 catch 块
- LLM Code Smells: 占位实现、过度泛化抽象
- Impact Analysis: 破坏性变更、API 签名修改
- Simplification: 重复逻辑、不必要复杂度
- 控制流: if 嵌套 >3 层、for 嵌套 >2 层、循环内查询 (N+1)

---

### 3. Code Fixer

自动修复代码以符合编码规范。

| 触发词 | 说明 |
|--------|------|
| `/code-fixer`、"修复代码"、"fix code"、"规范化代码" | 自动修复 |

**修复分类：**

| 类型 | AUTO (自动修复) | CONFIRM (需确认) | SKIP (禁止) |
|------|----------------|-----------------|-------------|
| 格式 | 换行、缩进、空格 | - | - |
| 重构 | 冗余 else、early return | 方法拆分、if 嵌套优化 | - |
| 缺失 | @Override、defer Close | 新增构造函数 | - |
| 命名 | - | - | 变量名、函数名 |

**重要原则：** 绝对禁止修改用户定义的变量名、函数名、类名。

---

### 4. Code Simplifier

简化和优化代码，提升清晰度、一致性和可维护性。

| 触发词 | 说明 |
|--------|------|
| `/code-simplifier`、"简化代码"、"优化代码"、"重构代码"、"清理代码" | 代码简化 |

**支持语言：** Go、Java、Python

**核心原则：**
1. 保持功能不变
2. 提升清晰度（选择清晰而非简短）
3. 避免过度简化
4. 聚焦范围（默认只优化最近修改的代码）

---

### 5. Unit Test

自动检测项目语言，生成符合最佳实践的单元测试。

| 触发词 | 说明 |
|--------|------|
| `/unit-test`、"帮我写单元测试"、"写测试"、"添加单元测试" | 生成单元测试 |

**支持的语言和框架：**

| 语言 | 检测文件 | 测试框架 | 风格 |
|------|---------|---------|------|
| Go | `go.mod` | Mockey + Testify | Table-Driven Tests |
| Java | `pom.xml` / `build.gradle` | Spock 或 JUnit 5 | BDD / Given-When-Then |

**Java Spock 版本选择：**

| 编译版本 | Spock 版本 | Groovy GroupId |
|---------|-----------|----------------|
| Java 8/11 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 17+ | `2.4-M4-groovy-4.0` | `org.apache.groovy` |

**Go 测试命令：**
```bash
go test -gcflags="all=-l -N" -v ./...
```

---

### 6. UI/UX Pro Max

UI/UX 设计智能助手，提供全面的设计指南。

| 触发词 | 说明 |
|--------|------|
| `/ui-ux-pro-max`、"设计 UI"、"Review UX"、"生成配色"、"创建着陆页" | 设计智能 |

**功能特性：**
- 50+ UI 风格（glassmorphism、minimalism、brutalism 等）
- 97+ 配色方案（按产品类型分类）
- 57+ 字体搭配（Google Fonts）
- 99+ UX 指南（可访问性、动画、布局）
- 25+ 图表类型
- 9 个技术栈支持

**支持的技术栈：**
`html-tailwind`、`react`、`nextjs`、`vue`、`svelte`、`swiftui`、`react-native`、`flutter`、`shadcn`、`jetpack-compose`

**工作流：**
1. 分析需求 → 2. 生成 Design System (`--design-system`) → 3. 补充详细搜索 → 4. 获取技术栈指南

**命令示例：**
```bash
python3 skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness" --design-system -p "Project Name"
```

---

### 7. Skill Creator

创建和打包高质量的 Claude Code Skill。

| 触发词 | 说明 |
|--------|------|
| `/skill-creator`、"创建 skill"、"新建 skill"、"打包 skill" | Skill 创建 |

**Skill 结构：**
```
skill-name/
├── SKILL.md (required)     # 主文件，包含 YAML frontmatter 和 Markdown 指南
├── scripts/                # 可执行脚本
├── references/             # 参考文档
└── assets/                 # 输出资源（模板、图标等）
```

**创建流程：**
1. 理解 skill 用例 → 2. 规划内容 → 3. 初始化 (`init_skill.py`) → 4. 编辑 → 5. 打包 (`package_skill.py`)

---

### 8. Add or Update Skill

管理 Claude 和 Gemini 的 skill 同步。

| 触发词 | 行为 |
|--------|------|
| "添加 skill"、"add skill" | 将指定文件夹添加到两个目录（已存在则跳过）|
| "更新 skill"、"update skill"、"同步 skill" | 同步更新，自动处理单边存在的情况 |

**目标目录：**
- Claude: `~/.claude/skills/`
- Gemini: `~/.gemini/antigravity/skills/`
- 项目本地: `./skills/`

---

## 📁 workflow/ - Workflow 文档

独立的工作流定义文档，可以直接复制到项目中使用。

```
workflow/
├── plan-init.md            # 初始化工作流
├── plan-next.md            # 任务执行工作流 (TDD 循环)
├── plan-log.md             # 手动日志工作流
├── plan-archive.md         # 归档工作流
├── code-review.md          # 代码审查工作流
├── code-fixer.md           # 代码自动修复工作流
├── unit-test.md            # 单元测试工作流
└── agent.md                # Agent 完整工作流文档 (汇总)
```

### Agent 工作流 (plan-*.md)

结构化的 TDD 开发工作流，核心特性：

- **抗遗忘**：通过任务日志恢复上下文
- **抗范围蔓延**：JSON 定义范围，日志提供详情
- **任务隔离**：每个任务独立日志文件 (`logs/task-{id}.log`)
- **TDD 强制**：必须先 RED 再 GREEN

**执行流程：**
```
READ → EXPLORE → PLAN → RED 🔴 → IMPLEMENT → GREEN 🟢 → COMMIT
```

### code-review.md - 代码审查工作流

基于 diff 的代码审查，自动检测语言并应用对应规范：
- **Java**：阿里巴巴 Java 开发规范
- **Go**：字节跳动 Go 开发规范
- **Frontend**：React/TypeScript 最佳实践
- **Backend**：Python/FastAPI 最佳实践

### code-fixer.md - 代码自动修复工作流

自动修复代码规范问题：
- **AUTO**：小问题自动修复（格式、注解、defer Close）
- **CONFIRM**：大改动需确认（方法拆分、新增构造函数）
- **SKIP**：禁止修改用户定义的变量名

### unit-test.md - 单元测试工作流

自动检测项目类型，生成符合最佳实践的单元测试：

- **Go**：Table-Driven Tests + Mockey + Testify
- **Java**：Spock (BDD) 或 JUnit 5 + Mockito + AssertJ

---

## 📁 common/ - 共享参考文档

可复用的测试指南和工作流定义。

```
common/
├── spock-test-guide.md     # Spock 单元测试完整指南 (中文)
├── go_test_spock.md        # Go 测试指南 (Mockey + Testify)
└── workTeam.md             # WorkTeam 角色分工工作流
```

### spock-test-guide.md

Java/Groovy 项目的 Spock BDD 测试指南：
- 基础结构 (given-when-then)
- Mock 对象操作
- 数据驱动测试 (where 块)
- 异常测试
- Maven 依赖配置

### go_test_spock.md

Go 项目的单元测试指南（字节跳动风格）：
- Table-Driven Tests 结构
- Mockey 运行时 Mock
- Testify 断言
- 必须的编译参数 `-gcflags="all=-l -N"`

### workTeam.md

5 人角色分工的产品开发流水线：

| 角色 | 命令 | 职责 |
|------|------|------|
| PM | `/pm` | 收集需求，输出 `prd.md` |
| UI 设计师 | `/ui` | 设计提示词，输出 `ui-prompts.md` |
| Nano Banana | `/nano` | 生成 UI 图片到 `assets/` |
| 前端工程师 | `/fe` | 构建前端界面 |
| 全栈工程师 | `/full` | 后端开发和迭代 |

---

## 快速开始

### 1. 使用 Agent 开发

```bash
# 1. 初始化项目
/plan-init

# 2. 定义任务后，开始执行
/plan-next

# 3. 继续下一个任务
/plan-next

# 4. 里程碑完成，归档
/plan-archive
```

### 2. 生成单元测试

```bash
# 自动检测语言并生成测试
/unit-test

# 或直接说
"帮我给 OrderService 写单元测试"
```

### 3. 代码质量工具

```bash
# 代码审查（需要先提供 diff）
git diff HEAD~1 | /code-review

# 自动修复代码规范
/code-fixer

# 简化优化代码
/code-simplifier
```

### 4. UI/UX 设计

```bash
# 生成设计系统
/ui-ux-pro-max
# 或
"设计一个 SaaS Dashboard"

# 命令行方式
python3 skills/ui-ux-pro-max/scripts/search.py "fintech saas" --design-system -p "Project"
```

### 5. Skill 管理

```bash
# 创建新 skill
/skill-creator

# 同步 skill 到 Claude/Gemini
/add_or_update_skill
# 然后说 "添加 skill" 或 "更新 skill"
```

### 6. 运行测试

**Go:**
```bash
go test -gcflags="all=-l -N" -v ./...
```

**Java (Maven):**
```bash
mvn test
```

**Java (Gradle):**
```bash
./gradlew test
```

---

## 设计原则

1. **资深开发视角**：考虑复用性、扩展性、健壮性
2. **精准执行**：只改该改的，不碰不该碰的
3. **TDD 优先**：测试驱动开发，先 RED 后 GREEN
4. **上下文恢复**：日志设计支持新会话快速恢复
5. **Token 高效**：任务级日志隔离，减少上下文消耗

---

## 文件说明

| 文件 | 用途 |
|------|------|
| `CLAUDE.md` | Claude Code 读取的项目指南，定义命令和规范 |
| `.claudeignore` | Claude Code 忽略的文件（.git, .DS_Store, *.skill） |
| `.gitignore` | Git 忽略的文件（构建产物、依赖、敏感信息） |

---

## 许可证

MIT License

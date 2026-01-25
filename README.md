# AI Messy Project

Claude Code Skills 和 Workflow 工具集，提供结构化的 TDD 开发工作流和多语言单元测试支持。

## 项目概览

本项目是一个 **Claude Code 增强工具集**，包含：

- **Agent 工作流**：基于 TDD 的结构化开发流程，支持任务管理、日志隔离、上下文恢复
- **Unit Test 工作流**：自动检测项目语言，生成符合最佳实践的单元测试
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
│   └── *.md                # 语言特定检查清单
├── code-fixer/             # 代码自动修复
│   ├── SKILL.md
│   └── references/         # 语言特定修复规则
├── unit-test/              # 单元测试生成
│   ├── SKILL.md
│   └── references/         # 测试参考文档
│       ├── go-mockey-testify.md    # Go 测试指南
│       ├── java-spock.md           # Spock 测试指南
│       ├── java-junit.md           # JUnit 5 测试指南
│       └── java-dependencies.md    # Java 依赖配置
├── code-simplifier/        # 代码简化优化
│   └── SKILL.md
├── skill-creator/          # Skill 创建指南
│   └── SKILL.md
├── ui-ux-pro-max/          # UI/UX 设计智能
│   └── SKILL.md
├── add_or_update_skill/    # Skill 管理工具
│   └── SKILL.md
└── *.skill                 # 打包后的技能文件 (ZIP 格式)
```

### Agent 命令

| 命令 | 用途 |
|------|------|
| `/plan-init` | 初始化项目，创建 `features.json` 和 `logs/` 目录 |
| `/plan-next` | 执行下一个任务，遵循 TDD 循环 (RED → GREEN → COMMIT) |
| `/plan-log` | 手动记录架构决策、紧急修复等非任务进度 |
| `/plan-archive` | 归档已完成工作到 `archives/YYYY-MM-DD-HHMMSS/` |

### Code Review & Fixer 命令

| 命令 | 用途 |
|------|------|
| `/code-review` | 审查代码变更，生成审查报告 |
| `/code-fixer` | 自动修复代码规范问题 |

### Unit Test 命令

| 命令 | 用途 |
|------|------|
| `/unit-test` | 自动检测语言，生成单元测试 |

**触发词：** "帮我写单元测试"、"写测试"、"添加单元测试"

**支持的语言：**

| 语言 | 检测文件 | 测试框架 | 风格 |
|------|---------|---------|------|
| Go | `go.mod` | Mockey + Testify | Table-Driven Tests |
| Java | `pom.xml` / `build.gradle` | Spock 或 JUnit 5 | BDD / Given-When-Then |

**设计原则：**
- **自动检测**：根据项目文件自动选择测试框架
- **风格统一**：匹配项目现有测试风格
- **编译版本优先**：Java 依赖版本由编译版本决定，而非运行时

### Code Simplifier 命令

| 命令 | 用途 |
|------|------|
| `/code-simplifier` | 简化和优化代码，提升清晰度和可维护性 |

**支持的语言：** Go、Java、Python

**触发词：** "简化代码"、"优化代码"、"重构代码"、"清理代码"

### UI/UX Design Intelligence

| 命令 | 用途 |
|------|------|
| `/ui-ux-pro-max` | UI/UX 设计智能助手 (设计系统、配色、排版、可访问性检查) |

**功能：**
提供 50+ 风格、97+ 配色方案、57+ 字体搭配的全面设计指南。支持从 "分析需求" 到 "生成 Design System" 再到 "具体实现" 的全流程。

**触发词：** "设计 UI"、"Review UX"、"生成配色"、"创建着陆页"

### Skill Creator

| 命令 | 用途 |
|------|------|
| `/skill-creator` | 创建和打包高质量的 Claude Code Skill |

**功能：**
提供 Skill 创建的最佳实践指南、模板初始化脚本 (`init_skill.py`) 和打包校验工具 (`package_skill.py`)。

**触发词：** "创建 skill"、"新建 skill"、"打包 skill"

### Skill 管理命令

| 命令 | 用途 |
|------|------|
| `/add_or_update_skill` | 添加或同步 skill 到 `~/.claude/skills` 和 `~/.gemini/antigravity/skills` |

**操作说明：**

| 触发词 | 行为 |
|--------|------|
| "添加 skill" | 将指定文件夹添加到两个目录（已存在则跳过）|
| "更新 skill" | 同步更新，自动处理单边存在的情况 |

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

### 3. 运行测试

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

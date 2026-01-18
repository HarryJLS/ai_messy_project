# Code Fixer - 代码自动修复工作流

基于 diff 的代码自动修复工作流，自动检测项目语言并应用对应规范修复代码。

## 核心概念

| 语言 | 检测方式 | 规范 |
|------|---------|------|
| Java | `*.java` | 阿里巴巴 Java 开发规范 |
| Go | `*.go` | 字节跳动 Go 开发规范 |
| Frontend | `*.tsx`, `*.jsx` | React/TypeScript 规范 |
| Python | `*.py` | FastAPI/异步 规范 |

**设计原则**：
- **禁止改名**：绝对不修改用户定义的变量名、函数名、类名
- **小改自动**：格式、缩进、冗余代码等小问题直接修复
- **大改确认**：方法拆分、新增构造函数等结构性改动需用户确认
- **透明度高**：所有修复都在报告中列出

---

## 修复分类

| 类型 | 行为 | 示例 |
|------|------|------|
| **AUTO** | 自动修复，无需确认 | 格式化、添加注解、defer Close |
| **CONFIRM** | 需用户确认后修复 | 方法拆分、新增构造函数 |
| **SKIP** | 禁止修改，仅报告 | 变量名、函数名、类名 |

---

## 工作流: code-fixer

**指令**: `/code-fixer`

**描述**: 自动修复代码中的规范问题

### 阶段 1: 获取 Diff

**默认行为**：直接执行 `git diff` 获取未提交的更改

```bash
git diff
```

**如果 `git diff` 为空**：询问用户选择其他来源

| 来源 | 命令 |
|------|------|
| 已暂存的更改 | `git diff --cached` |
| 与主分支对比 | `git diff main..HEAD` |
| 最近一次提交 | `git diff HEAD~1` |
| 指定文件 | 用户提供文件路径 |

⛔ **门控**: 必须获取到非空 diff 或明确的文件列表才能继续

---

### 阶段 2: 解析与检测

1. **解析 diff**：
   - 提取变更文件列表
   - 提取新增/修改的代码行
   - 记录行号范围

2. **检测语言/领域**：

```
文件扩展名检测：
├─ *.java → Java 领域 → 加载 Java 修复规则
├─ *.go → Go 领域 → 加载 Go 修复规则
├─ *.tsx/*.jsx → Frontend 领域 → 加载 Frontend 修复规则
├─ *.py → Backend 领域 → 加载 Backend 修复规则
└─ 其他 → 仅通用修复
```

3. **加载修复规则**：根据检测结果加载对应的修复规则

---

### 阶段 3: 问题分类

扫描 diff 中的问题，分为三类：

#### AUTO（自动修复）

| 类别 | 问题 |
|------|------|
| 通用 | 单行超长、冗余 else、空 catch 块 |
| Java | @Override、long L、日志占位符、switch default、大括号 |
| Go | gofmt、错误包装、defer Close/Unlock、slice/map 预分配、strings.Builder |
| Frontend | 列表 key 属性、index 作为 key |
| Backend | requests timeout、yaml.safe_load |

#### CONFIRM（需确认）

| 类别 | 问题 |
|------|------|
| 结构 | 方法/函数拆分（Go>50行、Java>70行） |
| Go | 新增构造函数、添加 context 参数 |
| 通用 | 抽取公共逻辑、架构调整 |

#### SKIP（禁止修改）

| 类别 | 说明 |
|------|------|
| 命名 | 变量名、函数名、类名、参数名 |

---

### 阶段 4: 执行自动修复

对 **AUTO** 类问题逐一修复：

1. 读取完整文件内容
2. 定位问题位置
3. 应用修复
4. 使用 Edit 工具写入
5. 记录修复内容

**修复示例**：

```go
// 修复前：冗余 else
if err != nil {
    return err
} else {
    return nil
}

// 修复后
if err != nil {
    return err
}
return nil
```

```java
// 修复前：空 catch
catch (Exception e) {}

// 修复后
catch (Exception e) {
    log.error("Error occurred", e);
}
```

---

### 阶段 5: 用户确认

对 **CONFIRM** 类问题汇总询问：

```
检测到以下需确认的改动：

1. [file.go:45] 函数 processData (68行) 超过 50 行限制
   建议：拆分为 processDataInit() + processDataCore()

2. [service.go:12] 检测到直接字段赋值
   建议：添加 NewUserService(repo, cache) 构造函数

3. [utils.go:30] 与 [utils.go:80] 存在重复逻辑
   建议：抽取为 formatError() 公共函数

请选择要执行的改动：
□ 1. 拆分 processData
□ 2. 添加构造函数
□ 3. 抽取公共函数
□ 全部执行
□ 全部跳过
```

⛔ **等待用户选择** 后执行对应修复

---

### 阶段 6: 生成报告

输出修复报告：

```markdown
## 修复完成

### 自动修复 (已完成) ✅
| 文件 | 行号 | 修复内容 |
|------|------|----------|
| service.go | 23 | 添加 defer Close |
| handler.go | 45-48 | 删除冗余 else |
| utils.go | 12 | 错误包装 fmt.Errorf |

### 用户确认 (已执行/已跳过)
| 文件 | 行号 | 改动 | 状态 |
|------|------|------|------|
| service.go | 10-80 | 函数拆分 | ✅ 已执行 |
| repo.go | 5 | 新增构造函数 | ⏭️ 已跳过 |

### 跳过 (禁止修改) ⚠️
| 文件 | 行号 | 原因 |
|------|------|------|
| model.go | 15 | 变量名 `userCnt` 建议改为 `userCount` |
| entity.java | 22 | 布尔属性 `isValid` 建议改为 `valid` |
```

---

## 完成后输出

```
✅ 代码修复完成！

📊 汇总:
• 自动修复: {N} 项
• 用户确认: {M} 项（执行 {X}，跳过 {Y}）
• 跳过（命名）: {K} 项

📝 变更文件:
• file1.go - 3 处修复
• file2.java - 2 处修复

⚠️ 命名建议（未修改）:
• model.go:15 - userCnt → userCount
• entity.java:22 - isValid → valid

下一步:
• 运行测试确认修复正确
• 使用 /code-review 检查是否还有遗漏
```

---

## 自动修复详细规则

### 通用规则

| 问题 | 修复方式 |
|------|---------|
| 单行 > 120 字符 | 在运算符/逗号后换行 |
| 冗余 else | 删除 else，提前 return |
| 空 catch/except | 添加日志记录 |

### Go 规则

| 问题 | 修复方式 |
|------|---------|
| 未 gofmt | 运行 `gofmt -w` |
| `return err` | 改为 `return fmt.Errorf("...: %w", err)` |
| Open 后无 Close | 添加 `defer f.Close()` |
| Lock 后无 Unlock | 添加 `defer mu.Unlock()` |
| append 循环无预分配 | 添加 `make([]T, 0, len(items))` |
| 字符串循环拼接 | 改用 `strings.Builder` |

### Java 规则

| 问题 | 修复方式 |
|------|---------|
| 缺少 @Override | 添加注解 |
| `100l` | 改为 `100L` |
| `log.info("x" + y)` | 改为 `log.info("x {}", y)` |
| switch 无 default | 添加 `default: break;` |
| if/else 无大括号 | 添加大括号 |

### Frontend 规则

| 问题 | 修复方式 |
|------|---------|
| map 无 key | 添加 `key={item.id}` |
| index 作为 key | 改用唯一 id |

### Backend 规则

| 问题 | 修复方式 |
|------|---------|
| requests 无 timeout | 添加 `timeout=30` |
| yaml.load | 改为 `yaml.safe_load` |

---

## 需确认改动详细规则

| 改动类型 | 触发条件 | 建议内容 |
|---------|---------|---------|
| 函数拆分 | Go>50行 / Java>70行 | 提供拆分方案 |
| 新增构造函数 | Go 直接字段赋值 | 生成 NewXxx 函数 |
| 添加 context | Go IO/RPC 函数无 ctx | 添加首参 |
| 抽取公共逻辑 | 重复代码块 | 提供函数签名 |

---

## 禁止修改项

⚠️ **以下内容绝对不修改，仅在报告中提示**：

- 变量名（即使不符合命名规范）
- 函数名
- 类名
- 参数名
- 常量名

**示例**：
```go
// 不会修改 userCnt，只会在报告中提示
userCnt := 10  // 建议: userCount
```

```java
// 不会修改 isValid，只会在报告中提示
private Boolean isValid;  // 建议: valid
```

---

## 与 code-review 的区别

| 方面 | code-review | code-fixer |
|------|-------------|------------|
| 目的 | 发现问题，生成报告 | 自动修复问题 |
| 输出 | 审查报告（只读） | 修改代码文件 |
| 命名问题 | 报告并建议修改 | 报告但禁止修改 |
| 结构问题 | 报告并建议 | 询问后可修复 |
| 格式问题 | 报告 | 直接修复 |

---

## 使用场景

| 场景 | 推荐工作流 |
|------|-----------|
| 提交前快速修复 | `/code-fixer` |
| PR 审查 | `/code-review` |
| 修复后验证 | `/code-fixer` → `/code-review` |
| 只看问题不修复 | `/code-review` |

---

## 快速命令

| 场景 | 命令 |
|------|------|
| 修复未提交更改 | `/code-fixer` |
| 修复最近提交 | `git diff HEAD~1` 后 `/code-fixer` |
| 修复指定文件 | 提供文件路径后 `/code-fixer` |

---

## 注意事项

1. **变量名绝对不改**：只在报告中提示，需用户手动修改
2. **保持功能不变**：修复只改形式，不改逻辑
3. **建议先 commit**：修复前 commit 现有代码，便于回滚
4. **增量修复**：每次修改后可运行测试验证
5. **透明度**：所有修复都在报告中列出，无隐藏改动

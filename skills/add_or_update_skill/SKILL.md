---
name: add_or_update_skill
description: 管理 skill 的添加和更新。将指定文件夹添加到 ~/.gemini/antigravity/skills 和 ~/.claude/skills，或同步更新这两个目录中的 skill。当用户说 "/add_or_update_skill"、"添加 skill"、"更新 skill"、"同步 skill" 时触发。
---

# Add or Update Skill

管理 Claude 和 Gemini 的 skill 同步。

## 目标目录

| 平台 | 路径 |
|------|------|
| Claude | `~/.claude/skills/` |
| Gemini | `~/.gemini/antigravity/skills/` |
| 项目本地 | `./skills/` (当前项目的 ai_messy_project/skills) |

## 协议

### 判断操作类型

根据用户输入判断执行哪种操作：

| 关键词 | 操作 |
|--------|------|
| "添加 skill"、"add skill" | 执行添加流程 |
| "更新 skill"、"update skill"、"同步 skill" | 执行更新流程 |

---

## 添加 Skill 流程

### 步骤 1: 获取源文件夹

1. **询问用户**：要添加哪个 skill 文件夹？
2. ⛔ **等待用户提供文件夹路径**

### 步骤 2: 验证源文件夹

检查用户提供的文件夹：

| 检查项 | 失败处理 |
|--------|----------|
| 文件夹存在 | 提示"文件夹不存在" |
| 包含 SKILL.md | 提示"缺少 SKILL.md，不是有效的 skill" |

### 步骤 3: 检查目标目录

检查 skill 是否已存在于目标目录：

```bash
ls ~/.claude/skills/{skill-name} 2>/dev/null
ls ~/.gemini/antigravity/skills/{skill-name} 2>/dev/null
```

**如果已存在**：
- 输出 "⏭️ {skill-name} 已存在于 {目录}，跳过"
- 不执行复制

### 步骤 4: 执行添加

对于不存在该 skill 的目录，执行复制：

```bash
cp -r {source-folder} ~/.claude/skills/
cp -r {source-folder} ~/.gemini/antigravity/skills/
```

### 步骤 5: 输出结果

```
✅ Skill 添加完成！

{skill-name}:
• ~/.claude/skills/{skill-name} - [已添加/已跳过(已存在)]
• ~/.gemini/antigravity/skills/{skill-name} - [已添加/已跳过(已存在)]
```

---

## 更新 Skill 流程

### 步骤 1: 获取 skill 名称

1. **询问用户**：要更新哪个 skill？
2. ⛔ **等待用户提供 skill 名称**

### 步骤 2: 检查 skill 存在情况

检查三个位置的 skill 状态：

```bash
ls ~/.claude/skills/{skill-name} 2>/dev/null && echo "claude: exists"
ls ~/.gemini/antigravity/skills/{skill-name} 2>/dev/null && echo "gemini: exists"
ls ./skills/{skill-name} 2>/dev/null && echo "local: exists"
```

### 步骤 3: 根据存在情况执行更新

| Claude | Gemini | Local | 操作 |
|--------|--------|-------|------|
| ✅ | ❌ | - | Claude → Gemini |
| ❌ | ✅ | - | Gemini → Claude |
| ✅ | ✅ | ✅ | Local → Claude, Local → Gemini |
| ✅ | ✅ | ❌ | 提示"两边都有，请指定更新源" |
| ❌ | ❌ | ✅ | Local → Claude, Local → Gemini |
| ❌ | ❌ | ❌ | 提示"❌ skill '{skill-name}' 在所有位置都不存在" |

**更新命令**：
```bash
# 删除旧版本并复制新版本
rm -rf {target}/{skill-name}
cp -r {source}/{skill-name} {target}/
```

### 步骤 4: 输出结果

```
✅ Skill 更新完成！

{skill-name}:
• 源: {source-path}
• ~/.claude/skills/{skill-name} - [已更新/已跳过]
• ~/.gemini/antigravity/skills/{skill-name} - [已更新/已跳过]
• ./skills/{skill-name} - [已更新/不存在/已跳过]
```

---

## 错误处理

| 错误 | 处理 |
|------|------|
| 目录不存在 | 自动创建: `mkdir -p {目录}` |
| 权限不足 | 提示用户检查权限 |
| skill 格式无效 | 提示缺少 SKILL.md |

⚠️ **操作完成后，停止并等待下一个命令。**

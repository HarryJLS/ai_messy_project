# Go 自动修复规则

基于字节跳动 Go 开发规范的自动修复项。

## AUTO 修复项

### 1. gofmt 格式化

**检测**: 代码未通过 gofmt
**修复**: 运行 `gofmt -w <file>`

### 2. 冗余 else 删除

**检测**:
```go
if condition {
    return x
} else {
    return y
}
```

**修复**:
```go
if condition {
    return x
}
return y
```

### 3. 错误包装

**检测**: `return err` 或 `return nil, err`
**修复**:
```go
return fmt.Errorf("描述操作失败: %w", err)
```

**注意**: 根据函数上下文生成描述，如 `failed to open file`

### 4. defer Close

**检测**:
```go
f, err := os.Open(path)
if err != nil {
    return err
}
// 无 defer f.Close()
```

**修复**:
```go
f, err := os.Open(path)
if err != nil {
    return err
}
defer f.Close()
```

适用于: `os.Open`, `os.Create`, `http.Get` 等返回需关闭资源的调用。

### 5. defer Unlock

**检测**:
```go
mu.Lock()
// ... 代码
mu.Unlock()
```

**修复**:
```go
mu.Lock()
defer mu.Unlock()
// ... 代码（删除原 Unlock 调用）
```

### 6. slice 预分配

**检测**:
```go
var result []int
for _, item := range items {
    result = append(result, item.Value)
}
```

**修复**:
```go
result := make([]int, 0, len(items))
for _, item := range items {
    result = append(result, item.Value)
}
```

### 7. map 预分配

**检测**: `make(map[K]V)` 后循环填充
**修复**: `make(map[K]V, len(items))`

### 8. strings.Builder

**检测**:
```go
var s string
for _, item := range items {
    s += item
}
```

**修复**:
```go
var builder strings.Builder
for _, item := range items {
    builder.WriteString(item)
}
s := builder.String()
```

### 9. HTTP Body 关闭

**检测**: `http.Get/Post/Do` 后无 `resp.Body.Close()`
**修复**:
```go
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close()
```

---

## CONFIRM 修复项

### 1. 函数拆分

**条件**: 函数超过 50 行
**确认内容**: 提供拆分方案，列出建议的子函数

### 2. 新增构造函数

**检测**:
```go
svc := &UserService{}
svc.repo = repo
svc.cache = cache
```

**建议**:
```go
func NewUserService(repo Repository, cache Cache) *UserService {
    return &UserService{
        repo:  repo,
        cache: cache,
    }
}
```

### 3. 添加 context 参数

**检测**: 函数涉及 IO/RPC 但第一个参数非 `ctx context.Context`
**建议**: 添加 `ctx context.Context` 作为第一个参数

### 4. 抽取公共函数

**检测**: 重复代码块（>5 行相似）
**建议**: 抽取为独立函数，提供函数签名

---

## SKIP 项 (禁止修改)

### 变量命名
即使不符合规范也不修改：
- `userCnt` 不改 `userCount`
- `Id` 不改 `ID`
- `Url` 不改 `URL`

**仅在报告中提示**:
```
[SKIP] service.go:15 变量 `userCnt` 建议改为 `userCount`（已跳过，不修改变量名）
```

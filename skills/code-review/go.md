# Go Review Checklist (字节跳动规范)

基于字节跳动 Go 语言编码规范的代码审查检查项。

添加以下分类到汇总表：
- 代码格式 (Go)
- 命名规范 (Go)
- 错误处理
- 并发安全
- 性能优化
- 控制流

---

## Category: 代码格式 (Go)

### 代码格式化

**检查方式**:
```bash
# 检查是否通过 gofmt 格式化
gofmt -d *.go
```

**通过标准**: 所有代码必须通过 gofmt 格式化
**严重级别**: High

---

### 函数行数限制

**检查方式**:
```bash
# 统计函数行数，找出超过 50 行的函数
awk '/^func /{start=NR; name=$0} /^\}/{if(start && NR-start>50) print FILENAME":"start": "name" ("NR-start" lines)"; start=0}' *.go
```

**通过标准**: 单个函数不超过 50 行
**严重级别**: Medium
**建议**: 超过 50 行的函数应拆分为多个小函数

---

### 单行字符数限制

**检查方式**:
```bash
# 找出超过 120 字符的行
awk 'length > 120 {print FILENAME":"NR": ("length" chars)"}' *.go
```

**通过标准**: 单行字符数不超过 120 个
**严重级别**: Low
**自动修复**: 在合适位置换行

```go
// 超过 120 字符时换行示例
result, err := someService.ProcessData(ctx, param1, param2,
    param3, param4)

// 长字符串分行
message := "This is a very long error message that exceeds " +
    "the maximum line length limit"
```

---

### 依赖注入使用构造函数

**搜索模式**:
```bash
# 直接赋值结构体字段（非构造函数注入）
grep -rE "^\s+\w+\.\w+\s*=\s*&?\w+\{" --include="*.go"

# 检查是否有 New 构造函数
grep -rE "^func\s+New\w+\(" --include="*.go"
```

**通过标准**: 依赖通过构造函数（NewXxx）注入，不直接赋值字段
**严重级别**: Medium

```go
// 推荐：构造函数注入
func NewUserService(repo UserRepository, cache Cache) *UserService {
    return &UserService{
        repo:  repo,
        cache: cache,
    }
}

// 不推荐：直接赋值字段
svc := &UserService{}
svc.repo = repo
svc.cache = cache
```

---

## Category: 命名规范 (Go)

### 包名规范

**搜索模式**:
```bash
# 包名含下划线或大写
grep -rE "^package\s+[A-Z_]" --include="*.go"
grep -rE "^package\s+\w*_\w*" --include="*.go"
```

**通过标准**: 包名全小写，不含下划线，简短有意义
**严重级别**: Medium

---

### 导出命名

**搜索模式**:
```bash
# 导出函数/类型未使用大驼峰
grep -rE "^func\s+[a-z]" --include="*.go" | grep -v "^func\s+(.*)\s+"
grep -rE "^type\s+[a-z]" --include="*.go"
```

**通过标准**: 导出的函数、类型、常量首字母大写
**严重级别**: Medium

---

### 缩写命名

**搜索模式**:
```bash
# ID/URL/HTTP 等缩写未全大写
grep -rE "\b(Id|Url|Http|Api|Sql|Json|Xml)\b" --include="*.go"
```

**通过标准**: 缩写词全大写（ID, URL, HTTP, API）
**严重级别**: Low

---

### 变量命名简洁

**检查要点**:
- 局部变量名简短
- 循环变量用单字母 i, j, k
- 接收者名用类型首字母小写

**通过标准**: 变量名简洁有意义，避免冗长
**严重级别**: Low

```go
// 推荐
for i, v := range items {}
func (s *Server) Start() {}

// 不推荐
for index, value := range items {}
func (server *Server) Start() {}
```

---

## Category: 错误处理

### 错误必须处理

**搜索模式**:
```bash
# 忽略错误返回值
grep -rE "\w+,\s*_\s*:?=\s*\w+\(" --include="*.go"
grep -rE "^\s*\w+\([^)]*\)\s*$" --include="*.go"
```

**通过标准**: 所有返回 error 的函数调用必须检查错误
**严重级别**: Critical

---

### 错误包装

**搜索模式**:
```bash
# 直接返回错误未包装
grep -rE "return\s+err\s*$" --include="*.go"
grep -rE "return\s+nil,\s*err\s*$" --include="*.go"
```

**通过标准**: 错误需用 fmt.Errorf + %w 包装，提供上下文
**严重级别**: Medium

```go
// 推荐
if err != nil {
    return fmt.Errorf("failed to process user %d: %w", userID, err)
}

// 不推荐
if err != nil {
    return err
}
```

---

### panic 使用限制

**搜索模式**:
```bash
# 业务代码中使用 panic
grep -rE "\bpanic\(" --include="*.go" | grep -v "_test.go\|main.go"
```

**通过标准**: panic 仅用于不可恢复的启动失败，业务代码禁用
**严重级别**: High

---

### recover 使用

**搜索模式**:
```bash
# recover 未在 defer 中
grep -rE "\brecover\(\)" --include="*.go" -B 3 | grep -v "defer"
```

**通过标准**: recover 只能在 defer 函数中使用
**严重级别**: High

---

## Category: 并发安全

### 数据竞争

**检查方式**:
```bash
# 使用 race detector
go test -race ./...
```

**通过标准**: 无数据竞争
**严重级别**: Critical

---

### goroutine 泄漏

**搜索模式**:
```bash
# 无限制启动 goroutine
grep -rE "go\s+func\(" --include="*.go" | grep -v "sync.WaitGroup\|context\|select"
```

**通过标准**: goroutine 必须有退出机制（context/channel/WaitGroup）
**严重级别**: High

---

### channel 未关闭

**搜索模式**:
```bash
# make(chan) 后未 close
grep -rE "make\(chan\s+" --include="*.go" -l | xargs grep -L "close\("
```

**通过标准**: 发送方负责关闭 channel
**严重级别**: Medium

---

### 锁使用

**搜索模式**:
```bash
# Lock 后未 defer Unlock
grep -rE "\.Lock\(\)" --include="*.go" -A 1 | grep -v "defer.*Unlock"
```

**通过标准**: Lock 后立即 defer Unlock
**严重级别**: High

```go
// 推荐
mu.Lock()
defer mu.Unlock()

// 不推荐
mu.Lock()
// ... 代码 ...
mu.Unlock()
```

---

### 原子操作

**搜索模式**:
```bash
# 简单计数器使用锁而非 atomic
grep -rE "sync\.Mutex" --include="*.go" -A 10 | grep -E "\+\+|--|\+="
```

**通过标准**: 单变量操作优先使用 atomic
**严重级别**: Low

---

## Category: 性能优化

### slice 预分配

**搜索模式**:
```bash
# append 循环未预分配
grep -rE "for.*\{" --include="*.go" -A 5 | grep "append\("
```

**通过标准**: 已知长度的 slice 应预分配容量
**严重级别**: Medium

```go
// 推荐
result := make([]int, 0, len(items))
for _, item := range items {
    result = append(result, item.Value)
}

// 不推荐
var result []int
for _, item := range items {
    result = append(result, item.Value)
}
```

---

### map 预分配

**搜索模式**:
```bash
# 循环中填充 map 未预分配
grep -rE "make\(map\[" --include="*.go" | grep -v ",\s*\d"
```

**通过标准**: 已知大小的 map 应预分配容量
**严重级别**: Low

```go
// 推荐
m := make(map[string]int, len(items))

// 不推荐
m := make(map[string]int)
```

---

### 字符串拼接

**搜索模式**:
```bash
# 循环中使用 + 拼接字符串
grep -rE "for.*\{" --include="*.go" -A 5 | grep -E '\+.*string|string.*\+'
```

**通过标准**: 多次拼接使用 strings.Builder
**严重级别**: Medium

```go
// 推荐
var builder strings.Builder
for _, s := range items {
    builder.WriteString(s)
}
result := builder.String()

// 不推荐
var result string
for _, s := range items {
    result += s
}
```

---

### 空 struct 占位

**搜索模式**:
```bash
# map 用于 set 时 value 非 struct{}
grep -rE "map\[.*\]bool" --include="*.go"
```

**通过标准**: 用 map 实现 set 时，value 使用 struct{}
**严重级别**: Low

---

## Category: 控制流

### 减少嵌套

**搜索技术**:
- 检查 if 嵌套超过 3 层
- 检查是否可用早返回减少嵌套

**通过标准**: 优先处理错误/特殊情况并尽早返回
**严重级别**: Medium

```go
// 推荐：早返回
func process(data *Data) error {
    if data == nil {
        return errors.New("data is nil")
    }
    if !data.Valid {
        return errors.New("data is invalid")
    }
    // 正常逻辑
    return nil
}

// 不推荐：深嵌套
func process(data *Data) error {
    if data != nil {
        if data.Valid {
            // 正常逻辑
            return nil
        }
        return errors.New("data is invalid")
    }
    return errors.New("data is nil")
}
```

---

### 冗余 else

**搜索模式**:
```bash
# if return 后有 else
grep -rE "return.*\n\s*\}\s*else\s*\{" --include="*.go"
```

**通过标准**: if 分支有 return，去掉 else
**严重级别**: Low

```go
// 推荐
if err != nil {
    return err
}
return nil

// 不推荐
if err != nil {
    return err
} else {
    return nil
}
```

---

## Category: 资源管理

### defer 关闭资源

**搜索模式**:
```bash
# Open 后未 defer Close
grep -rE "\.Open\(|os\.Open\(" --include="*.go" -A 3 | grep -v "defer.*Close"
```

**通过标准**: 打开的资源必须 defer 关闭
**严重级别**: High

---

### context 传递

**搜索模式**:
```bash
# 函数缺少 context 参数
grep -rE "^func\s+\w+\([^)]*\)" --include="*.go" | grep -v "ctx\s+context\.Context\|_test\.go"
```

**通过标准**: 涉及 IO/RPC 的函数第一个参数应为 context.Context
**严重级别**: Medium

---

### HTTP 响应体关闭

**搜索模式**:
```bash
# http.Get 后未关闭 Body
grep -rE "http\.(Get|Post|Do)\(" --include="*.go" -A 5 | grep -v "resp\.Body\.Close\|defer"
```

**通过标准**: HTTP 响应使用后必须关闭 Body
**严重级别**: High

```go
// 推荐
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close()
```

---

## Category: 测试

### 表驱动测试

**检查要点**:
- 测试用例使用 table-driven 风格
- 使用 t.Run 进行子测试

**通过标准**: 多场景测试使用表驱动方式
**严重级别**: Low

---

### 测试覆盖率

**检查方式**:
```bash
go test -cover ./...
```

**通过标准**: 核心逻辑测试覆盖率 > 60%
**严重级别**: Medium

---

# Java 自动修复规则

基于阿里巴巴 Java 开发手册的自动修复项。

## AUTO 修复项

### 1. @Override 注解

**检测**: 重写方法缺少 @Override
**修复**: 在方法声明前添加 `@Override`

### 2. long 字面量

**检测**: `100l`
**修复**: `100L`

### 3. 日志占位符

**检测**:
```java
log.info("User " + userId + " logged in");
```

**修复**:
```java
log.info("User {} logged in", userId);
```

适用于 debug/info/trace 级别。

### 4. switch default

**检测**: switch 块无 default 分支
**修复**:
```java
default:
    break;
```

或根据上下文添加异常：
```java
default:
    throw new IllegalStateException("Unexpected value: " + value);
```

### 5. if/else 大括号

**检测**:
```java
if (condition)
    doSomething();
```

**修复**:
```java
if (condition) {
    doSomething();
}
```

### 6. 空 catch 块

**检测**: `catch (Exception e) {}`
**修复**:
```java
catch (Exception e) {
    log.error("Error occurred", e);
}
```

### 7. 异常日志完整

**检测**:
```java
catch (Exception e) {
    log.error("Error: " + e.getMessage());
}
```

**修复**:
```java
catch (Exception e) {
    log.error("Error occurred", e);
}
```

### 8. 单行超长

**检测**: 超过 120 字符
**修复**: 在运算符、逗号、点号后换行

```java
// 修复前
String result = someService.processData(param1, param2, param3, param4, param5);

// 修复后
String result = someService.processData(param1, param2,
    param3, param4, param5);
```

---

## CONFIRM 修复项

### 1. 方法拆分

**条件**: 方法超过 70 行
**确认内容**: 提供拆分方案

### 2. 抽取常量

**检测**: 魔法值（非 0、1、-1 的数字）
**建议**: 抽取为常量

```java
// 检测
if (status == 3) { ... }

// 建议
private static final int STATUS_COMPLETED = 3;
if (status == STATUS_COMPLETED) { ... }
```

### 3. 线程池创建

**检测**: `Executors.newFixedThreadPool()` 等
**建议**: 改用 `ThreadPoolExecutor` 显式创建

### 4. SimpleDateFormat 线程安全

**检测**: static 或成员变量 SimpleDateFormat
**建议**: 改用 `DateTimeFormatter` 或 `ThreadLocal<SimpleDateFormat>`

### 5. 抽取公共方法

**检测**: 重复代码块
**建议**: 抽取为独立方法

---

## SKIP 项 (禁止修改)

### 变量命名
即使不符合规范也不修改：
- `_name` 不改 `name`
- `isValid` (POJO 布尔属性) 不改 `valid`
- `userId$` 不改 `userId`

**仅在报告中提示**:
```
[SKIP] User.java:25 布尔属性 `isValid` 建议改为 `valid`（已跳过，不修改变量名）
```

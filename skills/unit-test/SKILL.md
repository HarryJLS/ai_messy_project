
---
name: unit-test
description: Auto-detect project language and generate unit tests using appropriate patterns. Supports Go (Mockey + Testify) and Java (Spock or JUnit). Use when asked to write tests, create test cases, or add unit tests.
---

# Unit Test Skill

Auto-detect project language and generate unit tests using appropriate patterns and frameworks.

## Supported Languages

| Language | Detection | Framework | Reference |
|----------|-----------|-----------|-----------|
| Go | `go.mod` | Mockey + Testify (Table-Driven) | [go-mockey-testify.md](references/go-mockey-testify.md) |
| Java | `pom.xml` or `build.gradle` | Spock or JUnit 5 | [java-spock.md](references/java-spock.md) / [java-junit.md](references/java-junit.md) |

---

## Workflow

### Step 1: Language Detection

Check project root for build files:

```
┌─ go.mod exists?
│  └─ YES → Go Project → Use Mockey + Testify
│
├─ pom.xml exists?
│  └─ YES → Java (Maven) → Proceed to Java Flow
│
├─ build.gradle exists?
│  └─ YES → Java (Gradle) → Proceed to Java Flow
│
└─ None found → Ask user to specify language
```

### Step 2: Go Project Flow

**If Go project detected:**

1. Read `references/go-mockey-testify.md` for patterns
2. Generate tests using:
   - Table-Driven Tests (`[]struct` + `t.Run`)
   - Mockey for mocking (`mockey.Mock().To().Build()`)
   - Testify for assertions (`assert.Equal`, `assert.NoError`)
3. Remind user about required flags:
   ```bash
   go test -gcflags="all=-l -N" -v ./...
   ```

### Step 3: Java Project Flow

```
Detect Java project
    │
    ├─ Check for existing Spock
    │   - Search pom.xml/build.gradle for "spock-core"
    │   - Check for src/test/groovy/**/*.groovy files
    │
    │   ┌─ Has Spock?
    │   │  └─ YES → Use Spock style
    │   │          Read references/java-spock.md
    │   │
    │   └─ NO → Prompt user
    │
    └─ User prompt: "Add Spock dependencies?"
        │
        ├─ YES → Detect JDK version
        │        Add dependencies (see Step 3a)
        │        Use Spock style
        │
        └─ NO → Check for existing JUnit style
                Use JUnit 5 style
                Read references/java-junit.md
```

### Step 3a: JDK Version Detection (for Spock)

When adding Spock dependencies, detect **编译版本** (compile version):

> **重要**: 项目通常使用 Java 8 编译，但在 JDK 21 环境运行。Spock 依赖版本由**编译版本**决定，而非运行时版本。

**检测优先级:**

```
1. pom.xml / build.gradle 中的编译版本 (主要)
2. Dockerfile 中的运行时版本 (参考)
```

**Maven (`pom.xml`):**
```xml
<!-- 检查这些属性 -->
<java.version>1.8</java.version>
<!-- 或 -->
<maven.compiler.source>1.8</maven.compiler.source>
<maven.compiler.target>1.8</maven.compiler.target>
```

**Gradle (`build.gradle`):**
```groovy
sourceCompatibility = JavaVersion.VERSION_1_8
targetCompatibility = JavaVersion.VERSION_1_8
```

**Dockerfile (运行时环境参考):**
```dockerfile
# 常见模式: Java 8 编译 + JDK 21 运行
FROM openjdk:21-jdk-slim
# 或
FROM eclipse-temurin:21-jre
# 或 Java 8 编译 + JDK 8 运行
FROM openjdk:8-jdk-alpine
```

**版本选择规则:**

| 编译版本 | 运行时 | Spock 版本 | Groovy GroupId |
|---------|--------|-----------|----------------|
| Java 8 | JDK 8/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 11 | JDK 11/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 17+ | JDK 17/21 | `2.4-M4-groovy-4.0` | `org.apache.groovy` |

> **默认推荐**: 大多数项目使用 Java 8 编译，应使用 **Spock 2.3 + Groovy 3.x**

Read `references/java-dependencies.md` for full configuration.

### Step 4: Scan Existing Tests

Before generating tests, scan for existing test patterns:

```
src/test/
├─ groovy/  → Check for Spock patterns
├─ java/    → Check for JUnit patterns
└─ resources/
```

**Match existing style:**
- If project uses `@DisplayName` → Continue with that style
- If project uses Chinese descriptions in Spock → Continue with that style
- If project has specific assertion library → Use same library

---

## Test Generation Guidelines

### For All Languages

1. **Understand the code first**: Read the source file thoroughly
2. **Identify test scenarios**:
   - Happy path (normal flow)
   - Edge cases (null, empty, boundary values)
   - Error cases (exceptions, failures)
3. **Use meaningful test names**: Describe what is being tested and expected outcome
4. **Keep tests isolated**: Each test should be independent

### Go Specific

```go
func Test_MethodName(t *testing.T) {
    defer mockey.OffAll()  // Always clean up mocks

    type args struct { /* inputs */ }
    type mocks struct { /* mock behaviors */ }

    tests := []struct {
        name    string
        args    args
        mocks   mocks
        want    interface{}
        wantErr bool
    }{
        // Test cases here
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: Set up mocks
            // When: Call function
            // Then: Assert results
        })
    }
}
```

### Java Spock Specific

```groovy
def "测试方法描述"() {
    given: "准备测试数据"
    def input = new InputDTO(...)

    and: "模拟依赖行为"
    mockService.method(_) >> expectedValue

    when: "调用被测方法"
    def result = target.method(input)

    then: "验证结果"
    result == expected
    1 * mockService.save(_)  // Verify invocation
}
```

### Java JUnit Specific

```java
@Test
@DisplayName("测试描述")
void testMethodDescription() {
    // Given
    var input = new InputDTO(...);
    when(mockService.method(any())).thenReturn(expected);

    // When
    var result = target.method(input);

    // Then
    assertThat(result).isEqualTo(expected);
    verify(mockService, times(1)).save(any());
}
```

---

## User Interaction Points

### Prompt 1: Spock Installation

When Java project without Spock is detected:

```
检测到 Java 项目，但未发现 Spock 依赖。

是否添加 Spock 测试框架？
- Spock 提供更简洁的 BDD 风格语法
- 支持数据驱动测试（where 表格）
- 内置强大的 Mock 功能

[ 是，添加 Spock ] [ 否，使用 JUnit ]
```

### Prompt 2: Test Coverage Scope

When generating tests for a class with many methods:

```
检测到目标类有 [N] 个公共方法。

请选择测试覆盖范围：
[ 全部方法 ] [ 仅核心方法 ] [ 让我选择 ]
```

---

## Quick Reference

### Go Commands
```bash
# Run all tests with Mockey flags
go test -gcflags="all=-l -N" -v ./...

# Run specific test
go test -gcflags="all=-l -N" -v ./... -run TestFunctionName
```

### Java Commands
```bash
# Maven
mvn test
mvn test -Dtest=XxxServiceTest

# Gradle
./gradlew test
./gradlew test --tests XxxServiceTest
```

---

## Error Handling

### Go: Mock not working
- Ensure `-gcflags="all=-l -N"` flags are used
- Check that `defer mockey.OffAll()` is at the start

### Java: Spock tests not running
- Verify `gmavenplus-plugin` is configured
- Check that test files are in `src/test/groovy/`
- Ensure file names end with `Test.groovy` or `Spec.groovy`

### Java: Groovy version conflict
- **Java 8 编译** (无论运行时是 JDK 8 还是 21): 使用 `org.codehaus.groovy:groovy-all:3.0.x`
- **Java 17+ 编译**: 使用 `org.apache.groovy:groovy-all:4.0.x`
- 检查 Dockerfile 确认运行时环境，但依赖版本由编译版本决定

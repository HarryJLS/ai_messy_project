# Unit Test Workflow - 单元测试工作流

自动检测项目语言，使用合适的测试框架生成单元测试。

## 核心概念

| 语言 | 检测文件 | 测试框架 | 风格 |
|------|---------|---------|------|
| Go | `go.mod` | Mockey + Testify | Table-Driven Tests |
| Java | `pom.xml` / `build.gradle` | Spock 或 JUnit 5 | BDD / Given-When-Then |

**设计原则**：
- **自动检测**：根据项目文件自动选择测试框架
- **风格统一**：匹配项目现有测试风格
- **编译版本优先**：Java 依赖版本由编译版本决定，而非运行时
- **BDD 风格**：所有测试采用 Given-When-Then 结构

---

## 工作流 1: 语言检测

**指令**: `/unit-test` 或 "帮我写单元测试"

**描述**: 自动检测项目语言和测试框架

### 协议

#### 步骤 1: 检测项目类型

检查项目根目录的构建文件：

```
┌─ go.mod 存在?
│  └─ YES → Go 项目 → 进入 Go 工作流
│
├─ pom.xml 存在?
│  └─ YES → Java (Maven) → 进入 Java 工作流
│
├─ build.gradle 存在?
│  └─ YES → Java (Gradle) → 进入 Java 工作流
│
└─ 都不存在 → 询问用户指定语言
```

#### 步骤 2: 确认测试目标

询问用户：
```
请提供需要测试的代码：
1. 文件路径（如 src/main/java/com/example/Service.java）
2. 或类名/方法名
3. 或直接贴代码
```

⛔ **门控**: 等待用户提供测试目标

#### 步骤 3: 检测现有测试文件

**根据源文件路径，确定对应的测试文件位置：**

| 语言 | 源文件 | 测试文件 |
|------|--------|----------|
| Go | `foo/bar.go` | `foo/bar_test.go` |
| Java (JUnit) | `src/main/java/.../Service.java` | `src/test/java/.../ServiceTest.java` |
| Java (Spock) | `src/main/java/.../Service.java` | `src/test/groovy/.../ServiceSpec.groovy` |

**检测逻辑：**

```
检查测试文件是否存在?
├─ 存在 → 读取现有测试文件，在其基础上追加新测试方法
│         注意：保持现有测试风格和结构
│
└─ 不存在 → 创建新测试文件
```

> **重要**: 如果测试文件已存在，必须先读取其内容，在现有方法列表末尾追加新的测试方法，而非覆盖创建新文件。

---

## 工作流 2: Go 项目测试

**触发条件**: 检测到 `go.mod`

### 协议

#### 步骤 1: 读取源代码

1. 读取用户指定的 Go 文件
2. 分析函数签名、依赖关系
3. 识别外部依赖（需要 Mock 的部分）

#### 步骤 2: 生成测试代码

使用 **Table-Driven Tests + Mockey + Testify** 风格：

```go
func Test_MethodName(t *testing.T) {
    // 1. 全局清理 (Mockey 必须)
    defer mockey.OffAll()

    // 2. 定义测试数据结构
    type args struct {
        // 输入参数
    }
    type mocks struct {
        // Mock 行为控制
    }

    // 3. 定义测试用例表格 (对应 Spock 的 where 块)
    tests := []struct {
        name    string  // 测试场景描述（使用中文）
        args    args
        mocks   mocks
        want    interface{}
        wantErr bool
    }{
        {
            name: "正常流程_成功返回",  // 描述测试场景
            args: args{...},
            mocks: mocks{...},
            want: expected,
            wantErr: false,
        },
        {
            name: "异常流程_无效输入",  // 描述预期失败场景
            args: args{...},
            mocks: mocks{...},
            want: nil,
            wantErr: true,
        },
    }

    // 4. 执行循环
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: 准备测试数据和 Mock
            mockey.Mock(dependency.Method).To(func(...) ... {
                return tt.mocks.value
            }).Build()

            // When: 执行被测方法
            got, err := TargetFunc(tt.args.input)

            // Then: 验证返回结果
            if tt.wantErr {
                assert.Error(t, err)  // 验证返回错误
            } else {
                assert.NoError(t, err)  // 验证无错误
                assert.Equal(t, tt.want, got)  // 验证返回值匹配
            }
        })
    }
}
```

#### 步骤 3: 提醒运行命令

```
⚠️ Go Mockey 必须使用以下命令运行测试：

go test -gcflags="all=-l -N" -v ./...

参数说明：
- -l: 禁止内联 (disable inlining)
- -N: 禁止优化 (disable optimization)
```

### 测试场景覆盖

| 场景类型 | 示例 |
|---------|------|
| 正常流程 | 输入有效数据，返回期望结果 |
| 边界值 | 空值、零值、最大值、最小值 |
| 错误处理 | 外部依赖失败、无效输入 |
| 异常情况 | 并发、超时、资源不足 |

---

## 工作流 3: Java 项目测试

**触发条件**: 检测到 `pom.xml` 或 `build.gradle`

### 协议

#### 步骤 1: 检测现有测试框架

```
检查项目是否已有 Spock：
├─ pom.xml/build.gradle 中搜索 "spock-core"
├─ 检查 src/test/groovy/ 目录是否存在 .groovy 文件
│
├─ 已有 Spock?
│  └─ YES → 使用 Spock 风格
│
└─ 没有 Spock?
   └─ 询问用户
```

#### 步骤 2: 询问测试框架（如果没有 Spock）

```
检测到 Java 项目，但未发现 Spock 依赖。

是否添加 Spock 测试框架？
- Spock 提供更简洁的 BDD 风格语法
- 支持数据驱动测试（where 表格）
- 内置强大的 Mock 功能

[ 是，添加 Spock ] [ 否，使用 JUnit ]
```

⛔ **门控**: 等待用户选择

#### 步骤 3: 检测编译版本（如果选择 Spock）

> **重要**: 项目通常使用 Java 8 编译，但在 JDK 21 环境运行。
> Spock 依赖版本由**编译版本**决定，而非运行时版本。

**检测优先级**：
1. `pom.xml` / `build.gradle` 中的编译版本（主要）
2. `Dockerfile` 中的运行时版本（参考）

**Maven (`pom.xml`)**:
```xml
<java.version>1.8</java.version>
<!-- 或 -->
<maven.compiler.source>1.8</maven.compiler.source>
```

**Gradle (`build.gradle`)**:
```groovy
sourceCompatibility = JavaVersion.VERSION_1_8
```

**Dockerfile (运行时参考)**:
```dockerfile
FROM openjdk:21-jdk-slim   # Java 8 编译 + JDK 21 运行
FROM openjdk:8-jdk-alpine  # Java 8 编译 + JDK 8 运行
```

**版本选择规则**：

| 编译版本 | 运行时 | Spock 版本 | Groovy GroupId |
|---------|--------|-----------|----------------|
| Java 8 | JDK 8/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 11 | JDK 11/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 17+ | JDK 17/21 | `2.4-M4-groovy-4.0` | `org.apache.groovy` |

> **默认推荐**: 大多数项目使用 Java 8 编译，应使用 **Spock 2.3 + Groovy 3.x**

#### 步骤 4: 添加依赖（如果需要）

**Spock 依赖 (Java 8 编译)**：

```xml
<!-- Maven pom.xml -->
<properties>
    <spock.version>2.3-groovy-3.0</spock.version>
    <groovy.version>3.0.21</groovy.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.spockframework</groupId>
        <artifactId>spock-core</artifactId>
        <version>${spock.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>${groovy.version}</version>
        <type>pom</type>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>3.0.2</version>
            <executions>
                <execution>
                    <goals><goal>compileTests</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**JUnit 5 依赖**：

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.25.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 工作流 4: Spock 测试生成

**触发条件**: 用户选择 Spock 或项目已有 Spock

### 测试模板

```groovy
import spock.lang.Specification
import spock.lang.Unroll

class XxxServiceTest extends Specification {

    // 1. 声明 Mock 对象
    def xxxRepository = Mock(XxxRepository)
    def yyyService = Mock(YyyService)

    // 2. 创建被测对象
    def target = new XxxServiceImpl()

    // 3. 初始化依赖注入
    def setup() {
        target.xxxRepository = xxxRepository
        target.yyyService = yyyService
    }

    def "测试正常流程"() {
        given: "准备测试数据"
        def input = new InputDTO(id: 1L, name: "test")

        and: "模拟依赖行为"
        xxxRepository.getById(1L) >> new Entity(id: 1L)

        when: "调用被测方法"
        def result = target.doSomething(input)

        then: "验证结果"
        result != null
        result.id == 1L
    }

    @Unroll
    def "测试多种场景 - #description"() {
        given: "准备测试数据"
        def entity = new Entity(value: inputValue)

        when: "执行方法"
        def result = target.process(entity)

        then: "验证结果"
        result == expectedResult

        where: "测试数据表"
        description    | inputValue || expectedResult
        "正常值"       | 100        || true
        "边界值-零"    | 0          || false
        "边界值-负数"  | -1         || false
    }

    def "测试异常场景"() {
        given: "模拟异常触发条件"
        xxxRepository.getById(_) >> null

        when: "调用方法"
        target.process(invalidInput)

        then: "验证抛出异常"
        def ex = thrown(BusinessException)
        ex.message.contains("数据不存在")
    }
}
```

### Mock 速查表

| 场景 | 写法 |
|------|------|
| 创建 Mock | `def mock = Mock(Interface)` |
| 模拟返回值 | `mock.method(arg) >> value` |
| 模拟抛异常 | `mock.method(_) >> { throw new Ex() }` |
| 验证调用1次 | `1 * mock.method(_)` |
| 验证未调用 | `0 * mock.method(_)` |
| 任意参数 | `mock.method(_)` |
| 参数类型匹配 | `mock.method(_ as QueryDTO)` |
| 参数条件 | `mock.method({ it.id > 0 })` |

---

## 工作流 5: JUnit 5 测试生成

**触发条件**: 用户选择 JUnit 或项目已有 JUnit

### 测试模板

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.*;
import static org.assertj.core.api.Assertions.*;

class XxxServiceTest {

    @Mock
    private XxxRepository xxxRepository;

    @Mock
    private YyyService yyyService;

    private XxxServiceImpl target;
    private AutoCloseable mocks;

    @BeforeEach
    void setUp() {
        mocks = MockitoAnnotations.openMocks(this);
        target = new XxxServiceImpl(xxxRepository, yyyService);
    }

    @AfterEach
    void tearDown() throws Exception {
        mocks.close();
    }

    @Test
    @DisplayName("测试正常流程")
    void testNormalFlow() {
        // Given
        var input = new InputDTO(1L, "test");
        when(xxxRepository.getById(1L)).thenReturn(new Entity(1L));

        // When
        var result = target.doSomething(input);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
    }

    @ParameterizedTest(name = "{0}")
    @MethodSource("processTestCases")
    @DisplayName("测试多种场景")
    void testProcess(String description, int inputValue, boolean expectedResult) {
        // Given
        var entity = new Entity(inputValue);

        // When
        var result = target.process(entity);

        // Then
        assertThat(result).isEqualTo(expectedResult);
    }

    static Stream<Arguments> processTestCases() {
        return Stream.of(
            Arguments.of("正常值", 100, true),
            Arguments.of("边界值-零", 0, false),
            Arguments.of("边界值-负数", -1, false)
        );
    }

    @Test
    @DisplayName("测试异常场景")
    void testException() {
        // Given
        when(xxxRepository.getById(anyLong())).thenReturn(null);

        // When & Then
        assertThatThrownBy(() -> target.process(invalidInput))
            .isInstanceOf(BusinessException.class)
            .hasMessageContaining("数据不存在");
    }
}
```

### Mock 速查表

| 场景 | 写法 |
|------|------|
| Mock 对象 | `@Mock` 注解 |
| 初始化 Mock | `MockitoAnnotations.openMocks(this)` |
| 模拟返回值 | `when(mock.method(arg)).thenReturn(value)` |
| 模拟抛异常 | `when(mock.method(any())).thenThrow(new Ex())` |
| void方法抛异常 | `doThrow(ex).when(mock).voidMethod(any())` |
| 验证调用1次 | `verify(mock, times(1)).method(any())` |
| 验证未调用 | `verify(mock, never()).method(any())` |
| 任意参数 | `any()` 或 `any(Type.class)` |
| 参数条件 | `argThat(x -> x.getId() > 0)` |

---

## 工作流 6: 扫描现有测试风格

**描述**: 在生成测试前，扫描项目现有测试以匹配风格

### 协议

#### 步骤 1: 扫描测试目录

```
src/test/
├─ groovy/  → 检查 Spock 测试风格
├─ java/    → 检查 JUnit 测试风格
└─ resources/
```

#### 步骤 2: 提取风格特征

| 特征 | 检查内容 |
|------|---------|
| 命名风格 | 中文描述 vs 英文命名 |
| 断言库 | AssertJ vs JUnit 原生 vs Hamcrest |
| Mock 框架 | Mockito vs Spock 内置 |
| 数据驱动 | `@Unroll` + where 表 vs `@ParameterizedTest` |
| 注释风格 | `given:` 块 vs `// Given` 注释 |

#### 步骤 3: 匹配现有风格

- 如果项目使用 `@DisplayName` → 继续使用
- 如果项目使用中文描述 → 继续使用中文
- 如果项目有特定断言库 → 使用相同断言库

---

## 运行命令速查

### Go
```bash
# 运行所有测试 (Mockey 必须)
go test -gcflags="all=-l -N" -v ./...

# 运行单个测试
go test -gcflags="all=-l -N" -v ./... -run TestFunctionName
```

### Java Maven
```bash
# 运行所有测试
mvn test

# 运行单个测试类
mvn test -Dtest=XxxServiceTest

# 运行指定方法
mvn test -Dtest=XxxServiceTest#testMethodName
```

### Java Gradle
```bash
# 运行所有测试
./gradlew test

# 运行单个测试类
./gradlew test --tests XxxServiceTest
```

---

## 常见问题排查

### Go: Mock 不生效
- 确保使用 `-gcflags="all=-l -N"` 参数
- 检查 `defer mockey.OffAll()` 在函数开头

### Java: Spock 测试不运行
- 验证 `gmavenplus-plugin` 已配置
- 检查测试文件在 `src/test/groovy/` 目录
- 确保文件名以 `Test.groovy` 或 `Spec.groovy` 结尾

### Java: Groovy 版本冲突
- **Java 8 编译**（无论运行时）: `org.codehaus.groovy:groovy-all:3.0.x`
- **Java 17+ 编译**: `org.apache.groovy:groovy-all:4.0.x`
- 检查 Dockerfile 确认运行时环境，但依赖版本由编译版本决定

---

## 使用示例

### 示例 1: Go 项目

```
用户: 帮我给这个函数写单元测试
      [贴出 Go 代码]

AI: 检测到 Go 项目 (go.mod)
    使用 Mockey + Testify 风格

    [生成 Table-Driven 测试代码]

    ⚠️ 运行命令: go test -gcflags="all=-l -N" -v ./...
```

### 示例 2: Java 项目（新增 Spock）

```
用户: 帮我给 OrderService 写单元测试

AI: 检测到 Java (Maven) 项目
    未发现 Spock 依赖

    是否添加 Spock？ [是] [否]

用户: 是

AI: 检测到编译版本 Java 8 (pom.xml)
    运行时 JDK 21 (Dockerfile)
    推荐: Spock 2.3 + Groovy 3.x

    [提供 Maven 依赖配置]
    [生成 Spock 测试代码]
```

### 示例 3: Java 项目（使用 JUnit）

```
用户: 帮我写单元测试，用 JUnit

AI: 检测到 Java (Maven) 项目
    使用 JUnit 5 + Mockito + AssertJ

    [生成 JUnit 5 测试代码]
```

# JUnit 5 单元测试指南

> Native JUnit 5 测试风格，适用于不使用 Spock 的 Java 项目

## 1. 基础结构

```java
import org.junit.jupiter.api.*;
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
}
```

## 2. 测试方法结构

### 2.1 基础结构 (Given-When-Then 注释风格)

```java
@Test
@DisplayName("测试方法的中文描述")
void testMethodDescription() {
    // Given - 准备测试数据
    var input = new InputDTO(1L, "test");
    when(xxxRepository.getById(1L)).thenReturn(new Entity(1L));

    // When - 调用被测方法
    var result = target.doSomething(input);

    // Then - 验证结果
    assertThat(result).isNotNull();
    assertThat(result.getId()).isEqualTo(1L);
}
```

### 2.2 验证调用次数

```java
@Test
void testVerifyInvocations() {
    // Given
    when(xxxRepository.save(any())).thenReturn(new Entity(1L));

    // When
    target.process(new InputDTO());

    // Then
    verify(xxxRepository, times(1)).save(any());
    verify(yyyService, never()).delete(any());
    verify(xxxRepository, atLeast(1)).findById(anyLong());
}
```

## 3. Mock 对象操作

### 3.1 模拟返回值

```java
// 固定返回值
when(mockService.findById(1L)).thenReturn(new Entity(1L));

// 返回 null
when(mockService.findById(999L)).thenReturn(null);

// 返回列表
when(mockService.findAll()).thenReturn(List.of(entity1, entity2));

// 根据参数动态返回
when(mockService.process(anyString()))
    .thenAnswer(inv -> inv.getArgument(0, String.class).toUpperCase());

// 链式返回（多次调用返回不同值）
when(mockService.getNext())
    .thenReturn("first")
    .thenReturn("second")
    .thenReturn("third");
```

### 3.2 模拟抛出异常

```java
when(mockService.riskyOperation(any()))
    .thenThrow(new RuntimeException("error"));

// void 方法抛异常
doThrow(new RuntimeException("error"))
    .when(mockService).voidMethod(any());
```

### 3.3 参数匹配器

```java
// 任意参数
when(mockService.process(any())).thenReturn(result);

// 特定类型
when(mockService.process(any(String.class))).thenReturn(result);

// 任意 Long
when(mockService.findById(anyLong())).thenReturn(entity);

// 参数条件匹配
when(mockService.process(argThat(s -> s.length() > 5))).thenReturn(result);
```

## 4. 数据驱动测试 (Parameterized Tests)

### 4.1 @ParameterizedTest + @MethodSource

```java
@ParameterizedTest(name = "{0}")
@MethodSource("processTestCases")
@DisplayName("测试多种处理场景")
void testProcess(String description, int inputValue, int expectedResult) {
    // Given
    var entity = new Entity(inputValue);

    // When
    var result = target.process(entity);

    // Then
    assertThat(result).isEqualTo(expectedResult);
}

static Stream<Arguments> processTestCases() {
    return Stream.of(
        Arguments.of("正常值", 100, 200),
        Arguments.of("边界值-零", 0, 0),
        Arguments.of("边界值-负数", -1, -2)
    );
}
```

### 4.2 @ParameterizedTest + @CsvSource

```java
@ParameterizedTest(name = "输入 {0} 期望 {1}")
@CsvSource({
    "100, true",
    "0, false",
    "-1, false"
})
void testValidation(int input, boolean expected) {
    assertThat(target.isValid(input)).isEqualTo(expected);
}
```

### 4.3 @ParameterizedTest + @ValueSource

```java
@ParameterizedTest
@ValueSource(strings = {"", " ", "  "})
void testBlankStrings(String input) {
    assertThat(target.isBlank(input)).isTrue();
}
```

### 4.4 复杂数据结构

```java
@ParameterizedTest(name = "{0}")
@MethodSource("complexTestCases")
void testComplexScenario(
    String description,
    List<Map<String, Object>> inputData,
    int expectedSize
) {
    // Given
    var entities = inputData.stream()
        .map(map -> new Entity(
            (Long) map.get("id"),
            (String) map.get("name")
        ))
        .toList();

    // When
    var result = target.process(entities);

    // Then
    assertThat(result).hasSize(expectedSize);
}

static Stream<Arguments> complexTestCases() {
    return Stream.of(
        Arguments.of(
            "empty list",
            List.of(),
            0
        ),
        Arguments.of(
            "single item",
            List.of(Map.of("id", 1L, "name", "test")),
            1
        )
    );
}
```

## 5. 异常测试

### 5.1 验证抛出异常

```java
@Test
void testThrowsException() {
    // Given
    when(mockService.query(any())).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> target.process(invalidInput))
        .isInstanceOf(NullPointerException.class);
}
```

### 5.2 验证异常消息

```java
@Test
void testExceptionMessage() {
    // When & Then
    assertThatThrownBy(() -> target.process(invalidInput))
        .isInstanceOf(BusinessException.class)
        .hasMessageContaining("查询不到需要回撤的任务");
}
```

### 5.3 JUnit 5 原生方式

```java
@Test
void testExceptionNative() {
    var exception = assertThrows(
        BusinessException.class,
        () -> target.process(invalidInput)
    );

    assertThat(exception.getMessage()).contains("错误信息");
}
```

## 6. 断言方式 (AssertJ)

```java
// 基础断言
assertThat(result).isNotNull();
assertThat(result).isEqualTo(expected);
assertThat(result).isNotEqualTo(other);

// 集合断言
assertThat(list).hasSize(3);
assertThat(list).isEmpty();
assertThat(list).isNotEmpty();
assertThat(list).contains(element);
assertThat(list).containsExactly(e1, e2, e3);
assertThat(list).containsExactlyInAnyOrder(e3, e1, e2);
assertThat(list).extracting("name").contains("test");

// 字符串断言
assertThat(str).startsWith("prefix");
assertThat(str).endsWith("suffix");
assertThat(str).contains("middle");
assertThat(str).matches("regex.*pattern");

// 数值断言
assertThat(number).isGreaterThan(0);
assertThat(number).isBetween(1, 100);
assertThat(number).isCloseTo(10.0, within(0.1));

// 对象断言
assertThat(obj).hasFieldOrPropertyWithValue("name", "test");
assertThat(obj).usingRecursiveComparison().isEqualTo(expected);
```

## 7. 完整示例

```java
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private PaymentService paymentService;

    private OrderServiceImpl target;
    private AutoCloseable mocks;

    @BeforeEach
    void setUp() {
        mocks = MockitoAnnotations.openMocks(this);
        target = new OrderServiceImpl(orderRepository, paymentService);
    }

    @AfterEach
    void tearDown() throws Exception {
        mocks.close();
    }

    @Test
    @DisplayName("测试创建订单-正常流程")
    void testCreateOrderSuccess() {
        // Given
        var orderDTO = new OrderDTO(1L, BigDecimal.valueOf(100));
        when(orderRepository.save(any())).thenAnswer(inv -> {
            Order order = inv.getArgument(0);
            order.setId(1L);
            return order;
        });
        when(paymentService.charge(anyLong(), any())).thenReturn(true);

        // When
        var result = target.createOrder(orderDTO);

        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getStatus()).isEqualTo("CREATED");
        verify(orderRepository, times(1)).save(any());
        verify(paymentService, times(1)).charge(eq(1L), eq(BigDecimal.valueOf(100)));
    }

    @ParameterizedTest(name = "{0} -> {1} = {2}")
    @MethodSource("statusTransitionCases")
    @DisplayName("测试订单状态转换")
    void testCanTransition(String oldStatus, String newStatus, boolean canTransition) {
        // Given
        var order = new Order(1L, oldStatus);
        when(orderRepository.getById(1L)).thenReturn(order);

        // When
        var result = target.canTransition(1L, newStatus);

        // Then
        assertThat(result).isEqualTo(canTransition);
    }

    static Stream<Arguments> statusTransitionCases() {
        return Stream.of(
            Arguments.of("CREATED", "PAID", true),
            Arguments.of("CREATED", "CANCELLED", true),
            Arguments.of("PAID", "SHIPPED", true),
            Arguments.of("PAID", "CREATED", false),
            Arguments.of("CANCELLED", "PAID", false)
        );
    }

    @Test
    @DisplayName("测试支付失败抛出异常")
    void testCreateOrderPaymentFailed() {
        // Given
        when(paymentService.charge(anyLong(), any()))
            .thenThrow(new PaymentException("余额不足"));

        // When & Then
        assertThatThrownBy(() -> target.createOrder(new OrderDTO(1L, BigDecimal.valueOf(100))))
            .isInstanceOf(BusinessException.class)
            .hasMessageContaining("支付失败");
    }
}
```

## 8. 常用技巧速查

| 场景 | 写法 |
|------|------|
| Mock对象 | `@Mock` 注解 |
| 初始化 Mock | `MockitoAnnotations.openMocks(this)` |
| 模拟返回值 | `when(mock.method(arg)).thenReturn(value)` |
| 模拟抛异常 | `when(mock.method(_)).thenThrow(new Ex())` |
| void方法抛异常 | `doThrow(ex).when(mock).voidMethod(any())` |
| 验证调用1次 | `verify(mock, times(1)).method(any())` |
| 验证未调用 | `verify(mock, never()).method(any())` |
| 任意参数 | `any()` 或 `any(Type.class)` |
| 参数条件 | `argThat(x -> x.getId() > 0)` |
| 验证异常 | `assertThatThrownBy(...).isInstanceOf(Ex.class)` |
| 捕获异常 | `assertThrows(Ex.class, () -> ...)` |
| 数据驱动 | `@ParameterizedTest` + `@MethodSource` |
| 简单参数化 | `@CsvSource` 或 `@ValueSource` |
| 测试名称 | `@DisplayName("描述")` |

## 9. Maven 依赖配置

```xml
<properties>
    <junit.version>5.10.2</junit.version>
    <mockito.version>5.11.0</mockito.version>
    <assertj.version>3.25.3</assertj.version>
</properties>

<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>${mockito.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>${mockito.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>${assertj.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

## 10. 运行测试命令

```bash
# 运行所有测试
mvn test

# 运行单个测试类
mvn test -Dtest=XxxServiceTest

# 运行指定方法
mvn test -Dtest=XxxServiceTest#testMethodName

# Gradle
./gradlew test
./gradlew test --tests XxxServiceTest
```

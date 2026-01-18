# Java 测试依赖配置指南

> Spock 和 JUnit 5 的 Maven/Gradle 配置，适配不同编译版本

## 重要说明

> **编译版本 vs 运行时版本**
>
> 项目通常使用 **Java 8 编译**，但在 **JDK 21 环境运行**。
> Spock/Groovy 依赖版本由 **编译版本** 决定，而非运行时版本。
>
> - 编译版本: 看 `pom.xml` / `build.gradle`
> - 运行时版本: 看 `Dockerfile`

## 1. 版本检测

### 1.1 编译版本 (决定 Spock 版本)

**Maven (`pom.xml`):**
```xml
<properties>
    <!-- 常见写法 -->
    <java.version>1.8</java.version>
    <!-- 或 -->
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

**Gradle (`build.gradle`):**
```groovy
java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}
```

### 1.2 运行时版本 (参考)

**Dockerfile:**
```dockerfile
# 常见模式 1: Java 8 编译 + JDK 21 运行
FROM openjdk:21-jdk-slim
FROM eclipse-temurin:21-jre

# 常见模式 2: Java 8 编译 + JDK 8 运行
FROM openjdk:8-jdk-alpine
FROM eclipse-temurin:8-jre
```

---

## 2. Spock 配置 - Java 8 编译 (推荐)

### Maven 配置

```xml
<properties>
    <java.version>1.8</java.version>
    <!-- Spock 2.3 是最后支持 Groovy 3.x 的版本，兼容 JDK 8 -->
    <spock.version>2.3-groovy-3.0</spock.version>
    <groovy.version>3.0.21</groovy.version>
</properties>

<dependencies>
    <!-- Spock 核心框架 -->
    <dependency>
        <groupId>org.spockframework</groupId>
        <artifactId>spock-core</artifactId>
        <version>${spock.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Groovy 运行时 (Groovy 3.x 兼容 JDK 8) -->
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>${groovy.version}</version>
        <type>pom</type>
        <scope>test</scope>
    </dependency>

    <!-- JUnit Platform (Spock 2.x 必需) -->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-engine</artifactId>
        <version>1.9.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Groovy 编译插件 -->
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>3.0.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>compileTests</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- 测试运行插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Test.groovy</include>
                    <include>**/*Spec.groovy</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle 配置 (JDK 8)

```groovy
plugins {
    id 'java'
    id 'groovy'
}

java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}

dependencies {
    // Spock 2.3 + Groovy 3.x (JDK 8 兼容)
    testImplementation 'org.spockframework:spock-core:2.3-groovy-3.0'
    testImplementation 'org.codehaus.groovy:groovy-all:3.0.21'

    // JUnit Platform
    testImplementation 'org.junit.platform:junit-platform-engine:1.9.3'
}

test {
    useJUnitPlatform()
}
```

---

## 3. Spock 配置 - Java 17+ 编译

### Maven 配置

```xml
<properties>
    <java.version>21</java.version>
    <!-- Spock 2.4 + Groovy 4.x 支持 JDK 21 -->
    <spock.version>2.4-M4-groovy-4.0</spock.version>
    <groovy.version>4.0.21</groovy.version>
</properties>

<dependencies>
    <!-- Spock 核心框架 -->
    <dependency>
        <groupId>org.spockframework</groupId>
        <artifactId>spock-core</artifactId>
        <version>${spock.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Groovy 4.x (支持 JDK 21) -->
    <dependency>
        <groupId>org.apache.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>${groovy.version}</version>
        <type>pom</type>
        <scope>test</scope>
    </dependency>

    <!-- JUnit Platform (Spock 2.x 必需) -->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-engine</artifactId>
        <version>1.10.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-launcher</artifactId>
        <version>1.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Groovy 编译插件 -->
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>4.1.1</version>
            <executions>
                <execution>
                    <goals>
                        <goal>compileTests</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- 测试运行插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
            <configuration>
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Test.groovy</include>
                    <include>**/*Spec.groovy</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle 配置 (JDK 21)

```groovy
plugins {
    id 'java'
    id 'groovy'
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(21))
    }
}

dependencies {
    // Spock 2.4 + Groovy 4.x (JDK 21 支持)
    testImplementation 'org.spockframework:spock-core:2.4-M4-groovy-4.0'
    testImplementation 'org.apache.groovy:groovy-all:4.0.21'

    // JUnit Platform
    testImplementation 'org.junit.platform:junit-platform-engine:1.10.2'
    testImplementation 'org.junit.platform:junit-platform-launcher:1.10.2'
}

test {
    useJUnitPlatform()
}
```

---

## 4. JUnit 5 配置 (通用)

### Maven

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

### Gradle

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
    testImplementation 'org.mockito:mockito-core:5.11.0'
    testImplementation 'org.mockito:mockito-junit-jupiter:5.11.0'
    testImplementation 'org.assertj:assertj-core:3.25.3'
}

test {
    useJUnitPlatform()
}
```

---

## 5. 版本兼容性速查表

| 编译版本 | 运行时 | Spock 版本 | Groovy GroupId |
|---------|--------|-----------|----------------|
| Java 8  | JDK 8/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 11 | JDK 11/21 | `2.3-groovy-3.0` | `org.codehaus.groovy` |
| Java 17 | JDK 17/21 | `2.4-M4-groovy-4.0` | `org.apache.groovy` |
| Java 21 | JDK 21 | `2.4-M4-groovy-4.0` | `org.apache.groovy` |

> **注意**:
> - Groovy 4.x 的 groupId 从 `org.codehaus.groovy` 改为 `org.apache.groovy`
> - **大多数项目使用 Java 8 编译，应使用 Spock 2.3 + Groovy 3.x**

---

## 6. 检测项目是否已有 Spock

### Maven 项目
在 `pom.xml` 中搜索：
```xml
<artifactId>spock-core</artifactId>
```

### Gradle 项目
在 `build.gradle` 中搜索：
```groovy
spock-core
```

### 现有测试文件
检查是否存在 `.groovy` 测试文件：
```
src/test/groovy/**/*Test.groovy
src/test/groovy/**/*Spec.groovy
```

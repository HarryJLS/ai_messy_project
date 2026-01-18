# Go Unit Test Guide (Spock Style / ByteDance Spec)

> 本指南介绍如何在 Go 项目中实践类似 Spock 的 BDD 风格单元测试，采用字节跳动推荐的技术栈组合：**Table-Driven Tests** + **Mockey** + **Testify**。

## 1. 核心理念对比

这种写法将 Spock 的核心优势映射到 Go 的原生特性上，实现 **数据驱动** 与 **声明式 Mock**。

| 核心特性 | Spock (Groovy) | Go Implementation | 作用 |
| :--- | :--- | :--- | :--- |
| **测试场景** | `where:` 表格 | `[]struct` 切片定义 | 数据与逻辑分离，配置化管理用例 |
| **隔离执行** | `@Unroll` 注解 | `for` 循环 + `t.Run()` | 确保每个 Case 独立上下文 |
| **依赖模拟** | `Mock()` /Stub | `mockey.Mock().To()` | **运行时函数拦截**，无需生成代码，无需接口 |
| **断言** | `then:` 块 / `assert` | `testify/assert` | 提供人性化的断言与报错信息 |

## 2. 依赖引入

在项目中引入以下两个核心库：

```bash
# 字节自研 Mock 工具 (解决所有 Mock 难题)
go get github.com/bytedance/mockey

# 断言库 (提供更丰富的断言方法)
go get github.com/stretchr/testify
```

## 3. 标准模板结构

```go
func Test_MethodName(t *testing.T) {
    // 1. 全局清理 (对于 Mockey 必须)
    defer mockey.OffAll()

    // 2. 定义测试数据结构 (对应 Spock 的 where 表头)
    type args struct {
        // 输入参数字段
    }
    type mocks struct {
        // 定义 Mock 行为的控制开关或返回值
    }

    // 3. 定义测试用例表格
    tests := []struct {
        name    string  // 测试描述
        args    args
        mocks   mocks
        want    interface{} // 期望返回值
        wantErr bool        // 是否期望报错
    }{
        // Case 1: ...
        // Case 2: ...
    }

    // 4. 执行循环 (对应 @Unroll)
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: 设置 Mock
            mockey.Mock(SomeFunc).To(func(...) ... {
                return tt.mocks.someValue
            }).Build()

            // When: 执行被测函数
            got, err := TargetFunc(tt.args...)

            // Then: 断言结果
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
                assert.Equal(t, tt.want, got)
            }
        })
    }
}
```

## 4. 关键注意事项

> [!WARNING]
> COMPILATION FLAGS REQUIRED
>
> 这里的 **Mockey** 依赖二进制指令重写，运行时**必须**禁止内联和优化，否则 Mock 可能不生效或导致 Panic。

**运行/调试命令：**
```bash
go test -gcflags="all=-l -N" -v ./...
```
*   `-l`: 禁止内联 (disable inlining)
*   `-N`: 禁止优化 (disable optimization)

## 5. 完整实战示例

假设业务场景：支付服务需调用**风控系统**（RPC）和**数据库**（Dao），根据两者结果决定支付是否成功。

### 5.1 被测代码 (Service)
```go
package service

import (
    "errors"
    "example/dao"  // 假设的数据库包
    "example/risk" // 假设的风控包
)

func Pay(userID int64, amount float64) (string, error) {
    // 1. 外部依赖：风控检查
    if !risk.CheckRisk(userID) {
        return "", errors.New("risk rejected")
    }

    // 2. 外部依赖：余额查询
    balance, err := dao.GetBalance(userID)
    if err != nil {
        return "", err
    }

    // 3. 业务逻辑
    if balance < amount {
        return "", errors.New("insufficient balance")
    }

    return "TX_SUCCESS", nil
}
```

### 5.2 测试代码 (Test)

```go
package service

import (
    "errors"
    "testing"

    "github.com/bytedance/mockey"
    "github.com/stretchr/testify/assert"
    
    "example/dao"
    "example/risk"
)

func TestPay(t *testing.T) {
    // 确保测试结束后还原所有 Mock，避免污染其他测试
    defer mockey.OffAll()

    // 定义输入参数
    type args struct {
        userID int64
        amount float64
    }
    
    // 定义 Mock 行为控制
    type mocks struct {
        riskPass  bool    // 模拟风控结果
        balance   float64 // 模拟查到的余额
        dbError   error   // 模拟数据库错误
    }

    // 定义测试矩阵 (Spock's where block)
    tests := []struct {
        name    string
        args    args
        mocks   mocks
        want    string
        wantErr bool
        errText string // 额外的错误信息匹配
    }{
        {
            name: "Success_Normal",
            args: args{userID: 101, amount: 100},
            mocks: mocks{
                riskPass: true,
                balance:  200, // 余额充足
                dbError:  nil,
            },
            want:    "TX_SUCCESS",
            wantErr: false,
        },
        {
            name: "Fail_RiskRejected",
            args: args{userID: 102, amount: 100},
            mocks: mocks{
                riskPass: false,
                balance:  0,
                dbError:  nil,
            },
            want:    "",
            wantErr: true,
            errText: "risk rejected",
        },
        {
            name: "Fail_BalanceNotEnough",
            args: args{userID: 103, amount: 100},
            mocks: mocks{
                riskPass: true,
                balance:  50, // 余额不足
                dbError:  nil,
            },
            want:    "",
            wantErr: true,
            errText: "insufficient balance",
        },
        {
            name: "Fail_DBError",
            args: args{userID: 104, amount: 100},
            mocks: mocks{
                riskPass: true,
                balance:  0,
                dbError:  errors.New("db timeout"),
            },
            want:    "",
            wantErr: true,
            errText: "db timeout",
        },
    }

    // 执行测试循环
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: 注入 Mock 行为
            // Mock 风控函数
            mockey.Mock(risk.CheckRisk).To(func(uid int64) bool {
                return tt.mocks.riskPass
            }).Build()

            // Mock 数据库函数
            mockey.Mock(dao.GetBalance).To(func(uid int64) (float64, error) {
                return tt.mocks.balance, tt.mocks.dbError
            }).Build()

            // When: 调用被测方法
            got, err := Pay(tt.args.userID, tt.args.amount)

            // Then: 验证结果
            if tt.wantErr {
                assert.Error(t, err)
                if tt.errText != "" {
                    assert.Contains(t, err.Error(), tt.errText)
                }
            } else {
                assert.NoError(t, err)
                assert.Equal(t, tt.want, got)
            }
        })
    }
}
```

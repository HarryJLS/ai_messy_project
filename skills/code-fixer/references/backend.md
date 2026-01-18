# Backend 自动修复规则

Python/FastAPI 自动修复项。

## AUTO 修复项

### 1. HTTP 请求 timeout

**检测**:
```python
requests.get(url)
```

**修复**:
```python
requests.get(url, timeout=30)
```

适用于: requests.get/post/put/delete, httpx 调用

### 2. yaml.safe_load

**检测**:
```python
yaml.load(data)
```

**修复**:
```python
yaml.safe_load(data)
```

### 3. 空 except 块

**检测**:
```python
except Exception:
    pass
```

**修复**:
```python
except Exception as e:
    logger.error("Error occurred", exc_info=e)
```

### 4. bare except

**检测**:
```python
except:
    ...
```

**修复**:
```python
except Exception as e:
    ...
```

### 5. f-string 日志

**检测**:
```python
logger.info(f"User {user_id} logged in")
```

**修复**:
```python
logger.info("User %s logged in", user_id)
```

### 6. 类型注解补充

**检测**: 函数参数无类型注解
**修复**: 根据使用推断添加类型

```python
# 检测
def process(data):
    return data.strip()

# 修复
def process(data: str) -> str:
    return data.strip()
```

---

## CONFIRM 修复项

### 1. 函数拆分

**条件**: 函数超过 50 行
**确认内容**: 提供拆分方案

### 2. 依赖注入

**检测**: 路由中直接实例化服务
```python
@app.get("/users")
def get_users():
    service = UserService()  # 直接实例化
    return service.list()
```

**建议**: 使用 Depends()
```python
@app.get("/users")
def get_users(service: UserService = Depends(get_user_service)):
    return service.list()
```

### 3. Pydantic 模型

**检测**: dict 类型的请求体
**建议**: 创建 Pydantic 模型

### 4. async 改造

**检测**: sync 路由中使用阻塞 IO
**建议**: 改为 async + await

### 5. 抽取公共函数

**检测**: 重复代码块
**建议**: 抽取为独立函数

---

## SKIP 项 (禁止修改)

### 变量命名
即使不符合 PEP8 也不修改：
- 函数名
- 变量名
- 参数名
- 类名

**仅在报告中提示**:
```
[SKIP] service.py:15 变量 `userName` 建议改为 `user_name`（已跳过，不修改变量名）
```

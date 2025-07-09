# 📦 环境变量、异步文件与并发工具库使用文档

---

## 模块目录

| 模块名                | 说明 |
|-----------------------|------|
| `env_utils.py`        | 环境变量操作工具 |
| `async_file_utils.py` | 异步文件读写工具 |
| `concurrency_utils.py`| 并发任务调度工具 |

---

# 📁 1. `env_utils.py` — 环境变量管理工具

### ✅ 类：`EnvMethod`

#### 🔧 `newEnv(key: str, value: str, remarks: str = "", mark: str = "")`
- **功能**：添加新的环境变量并写入 `config.sh`
- **说明**：可选添加备注和分组标识
- **示例**：
```python
EnvMethod.newEnv("API_KEY", "12345", remarks="OpenAI 密钥", mark="基础配置")
```
- **结果**：
```bash
# 基础配置
export API_KEY="12345" # OpenAI 密钥
```

---

#### 🔧 `upEnv(key: str, value: str)`
- **功能**：更新指定的环境变量值
- **示例**：
```python
EnvMethod.upEnv("API_KEY", "67890")
```

---

#### 🔧 `readEnv(...)`
- **功能**：读取并转换环境变量（支持 int、list、bool 自动判断）
- **参数**：
  - `key`: 环境变量名
  - `default`: 默认值（变量为空时返回）
  - `codeInt`: 是否强转为整数或整数列表
  - `codeList`: 是否强转为列表
- **示例**：
```python
port = EnvMethod.readEnv("PORT", default=8000, codeInt=True)
flags = EnvMethod.readEnv("FLAGS", default=["a", "b"], codeList=True)
```

---

#### 🔧 `checkEnv(obj_self, config: List[Dict], err_exit: bool = True)`
- **功能**：按配置批量注入环境变量到对象属性
- **示例**：
```python
class Config: pass
EnvMethod.checkEnv(Config(), [
    {"env_key": "API_KEY", "attribute": "api_key", "type": str},
    {"env_key": "TIMEOUT", "attribute": "timeout", "type": int, "default": 10}
])
```

---

#### 🔧 `load_config_from_env(obj_self, config_obj, err_exit=True)`
- **功能**：通过类型注解 + `env_map` 映射自动注入属性值
- **配置类定义示例**：
```python
class AppConfig:
    env_map = {"api_key": "API_KEY", "timeout": "TIMEOUT"}
    api_key: str = ""
    timeout: int = 5

config = AppConfig()
EnvMethod.load_config_from_env(config, config)
```

---

# 📁 2. `async_file_utils.py` — 异步文件工具

### ✅ 类：`FileMethod`

#### 🔧 `read_str(file_name: str, mode: str = "r")`
- **说明**：读取文本内容（异步）
- **示例**：
```python
text = await FileMethod.read_str("README.md")
```

---

#### 🔧 `write_str(file_name: str, data: str, mode: str = "a")`
- **说明**：异步写入文本（自动加锁，避免并发冲突）
- **示例**：
```python
await FileMethod.write_str("log.txt", "日志内容
")
```

---

#### 🔧 `read_json(file_name: str)`
- **说明**：异步读取 JSON 文件并返回 Python 对象
- **示例**：
```python
config = await FileMethod.read_json("config.json")
```

---

#### 🔧 `write_json(file_name: str, updata: Dict | List | str, newBuild=False)`
- **说明**：更新 JSON 内容或新建写入文件
- **逻辑**：
  - `newBuild=True`：直接写入
  - `newBuild=False`：读取后合并再写入
- **示例**：
```python
await FileMethod.write_json("config.json", {"token": "abc"})
await FileMethod.write_json("list.json", ["entry"], newBuild=True)
```

---

#### 🔧 `write_image(file_name: str, image: bytes)`
- **说明**：将图片字节数据写入磁盘（异步）
- **示例**：
```python
await FileMethod.write_image("output.jpg", image_bytes)
```

---

# 📁 3. `concurrency_utils.py` — 并发任务执行工具

### ✅ 类：`RunMethod`

#### 🔧 `ConcRun(function, thread, *args, **kwargs)`
- **功能**：并发执行同一个函数多次
- **说明**：限制同时运行的任务数量
- **示例**：
```python
await RunMethod.ConcRun(my_func, 5, param="xx")
```

---

#### 🔧 `SubTaskConcRun(func, items: Any)`
- **说明**：提取类中的列表字段，依次并发执行
- **要求**：仅支持类中包含单个 list 属性
- **示例**：
```python
class A: urls = ["a", "b"]; token = "abc"
await RunMethod.SubTaskConcRun(fetch, A())
```

---

#### 🔧 `MultiTaskConcRun(func, items, thread, wait)`
- **说明**：将多个对象分批处理，每批线程数量限制，支持延迟
- **返回**：生成器（yield 多个批次）
- **示例**：
```python
async for result_batch in RunMethod.MultiTaskConcRun(fetch, tasks, thread=5, wait=1):
    ...
```

---

#### 🔧 `ReqConcRun(param: ReqConcParam)`
- **说明**：使用封装参数对象执行批量请求任务（封装函数和参数）
- **示例**：
```python
from concurrency_utils import ReqConcParam

param = ReqConcParam(func=my_func, task=[[x] for x in range(10)], thread=3)
async for res in RunMethod.ReqConcRun(param):
    ...
```

---

### 🧱 类：`ReqConcParam`
```python
ReqConcParam(
    func: Callable,
    task: List[Any],
    thread: int,
    wait: Union[int, float] = 0,
    **kwargs
)
```

### 🧱 类：`ReqConcResult`
```python
ReqConcResult(result: AsyncResponse, task: Any)
```

### 🧱 类：`RunMethod.MultiTaskConcRunResult`
```python
MultiTaskConcRunResult(response: List[AsyncResponse], cookie: Any)
```

---

# ✅ 说明与建议

- 所有涉及文件写入的操作都使用异步 `aiofiles` 实现，具备并发安全性。
- 环境变量读取时具备健壮的类型转换策略。
- 并发执行时建议控制线程数量，避免过载。
- 工具类适用于构建自动化平台、爬虫框架、异步微服务等应用场景。

---

© 本文档由 AI 自动生成，可嵌入至 MakerWorld 项目或 ReadMe 文档中。

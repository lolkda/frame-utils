# Python 高性能异步自动化工具库

这是一个专为自动化任务设计的Python工具库，具备高性能、异步、可扩展的特点。它内置了强大的HTTP客户端、代理管理、任务调度、设备UA生成等基础功能，并深度集成了 **京东(JD.com) API** 和 **青龙(Qinglong)面板** 的交互能力，是构建复杂自动化流程的理想选择。

## 核心功能 ✨

- **高性能HTTP客户端**: 基于 `aiohttp` 实现，具备自动重试和灵活的代理管理功能。
- **青龙面板无缝对接**: 强大的青龙OpenAPI客户端，可轻松通过代码管理面板中的环境变量、定时任务、配置文件等。
- **异步进程管理**: 非阻塞地执行和监控外部脚本（如 `.py`, `.js`, `.sh`），获取实时输出，并支持优雅地终止进程。
- **高级并发控制**: 提供易于使用的并发运行器，能够以指定并发数（线程）分批执行大量异步任务，并控制任务间的等待间隔。
- **异步安全的文件读写**: 提供带锁的异步文件操作方法，安全地读写字符串、JSON和图片文件，避免并发冲突。
- **灵活的任务调度器**: 支持与服务器时间同步，可实现高精度的定时、延时、周期性任务调度。
- **可扩展的通知系统**: 采用插件化设计，可以轻松集成多种推送渠道（如Telegram, Pushplus等），用于任务结果的实时通知。
- **强大的环境与配置管理**: 简化了环境变量的读取和类型转换，支持从 `config.sh` 文件加载配置。
- **丰富的User-Agent生成器**: 可生成覆盖苹果、华为、小米等主流品牌的上千种真实移动设备User-Agent，增强请求的模拟度。
- **结构化的日志系统**: 提供清晰、可定制的日志输出，支持按脚本和日期生成日志文件，便于调试和追踪。


---

## 快速开始 🚀

### 1. 项目配置 (`/env/config.sh`)

所有核心功能的配置都通过项目根目录下的 `/env/config.sh` 文件进行管理。在首次使用前，请务必根据以下模板创建并编辑此文件。

```shell
#!/bin/bash

# --- HTTP客户端与代理配置 ---
# 代理池API地址，支持pool(从API获取多个)或api(直接使用该地址)两种模式
export PROXY_URL="http://your_proxy_provider_api_url"
# 代理模式, "pool" 或 "api"
export PROXY_Method="pool"
# HTTP请求超时时间 (秒)
export Req_Timeout="15"
# 最大并发连接数
export Max_Connection="400"

# --- 京东API签名服务配置 ---
# 自建或第三方的签名服务API地址
export JD_SIGN_API="http://your_jd_sign_server/api/sign"
export JD_H5ST_API="http://your_jd_sign_server/api/h5st"

# --- 青龙面板OpenAPI配置 ---
# 可以配置多个青龙面板实例，通过前缀区分，例如 JD、ELM
# 第一个面板 (JD)
export OPENAPI_JD_NAME="我的JD面板"
export OPENAPI_JD_HOST="[http://192.168.1.10:5700](http://192.168.1.10:5700)"
export OPENAPI_JD_USER="your_client_id"
export OPENAPI_JD_PASSWORD="your_client_secret"

# 第二个面板 (ELM)
export OPENAPI_ELM_NAME="我的ELM面板"
export OPENAPI_ELM_HOST="[http://192.168.1.11:5700](http://192.168.1.11:5700)"
export OPENAPI_ELM_USER="another_client_id"
export OPENAPI_ELM_PASSWORD="another_client_secret"

# --- 推送通知插件配置 (以Telegram为例) ---
# 启用Telegram推送
export TELEGRAM_BOT_ISOPEN="true"
# Bot的Token
export TELEGRAM_BOT_TOKEN="123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11"
# 接收消息的用户或频道ID
export TELEGRAM_CHAT_ID="100000001"
# Telegram API代理 (可选)
export TELEGRAM_PROXY_URL="[http://127.0.0.1:7890](http://127.0.0.1:7890)"

# --- 日志等级配置 ---
# 10=DEBUG, 20=INFO, 30=WARNING, 40=ERROR
export log_level="20"
```

---

## 模块使用指南 🛠️

以下是各核心模块的详细使用方法和代码示例。

### **1. 青龙面板 API (`openApi.py`)**

`openApi.py` 模块提供了与青龙面板进行交互的全部能力，核心是 `openApiCommonMethod` (推荐) 和 `openApiMethod` (底层) 两个类。

#### **核心用法**
1.  **导入**: `from utils.openApi import openApiCommonMethod`。
2.  **实例化**: `ql_api = openApiCommonMethod()`。
3.  **指定面板**: `ql_api.openApi_Use = "PREFIX"`。这里的 `"PREFIX"` 必须与您在 `config.sh` 中配置的 `OPENAPI_PREFIX_...` 前缀一致 (如 "JD", "ELM")。
4.  **调用方法**: `await ql_api.some_method()`。

---
#### **`openApiCommonMethod` (高级封装 - 推荐使用)**

此类提供了最常用、最便捷的方法，隐藏了底层的API细节。

##### **`get_cookie(ContainerName, name)`**
快速获取指定面板中某个环境变量的所有值。
- `ContainerName`: 面板前缀，如 "JD"。
- `name`: 环境变量名，如 "JD_COOKIE"。
- **返回**: 一个包含所有同名环境变量值的列表 `List[str]`。

```python
cookies = await ql_api.get_cookie("JD", "JD_COOKIE")
print(f"获取到 {len(cookies)} 个JD_COOKIE。")
```

##### **`search_envs(keyword)`**
根据关键词模糊搜索环境变量。
- `keyword`: 搜索的关键词。
- **返回**: 包含环境变量信息字典的列表 `List[Dict]`。

##### **`update_envs(name, value, remarks, add_env, keyword)`**
更新或新增一个环境变量。它会先通过 `keyword` 搜索，如果找到就更新，找不到且 `add_env=True` 则新增。
- `name`, `value`, `remarks`: 环境变量的名称、值和备注。
- `add_env`: (可选, `bool`) 如果找不到是否新增。
- `keyword`: 用于定位要更新的环境变量的关键词。

```python
# 搜索并更新
await ql_api.update_envs(
    name="MY_TEST_VAR",
    value="new_value_123",
    remarks="由脚本更新",
    keyword="MY_TEST_VAR"
)
print("环境变量 MY_TEST_VAR 已更新。")

# 如果不存在则新增
await ql_api.update_envs(
    name="NEW_VAR",
    value="hello_world",
    remarks="新变量",
    add_env=True,
    keyword="NEW_VAR"
)
print("环境变量 NEW_VAR 已新增。")
```

##### **`disable_envs(id) / EnableEnvs(id)`**
禁用或启用一个或多个环境变量。
- `id`: 单个环境变量ID (`int`) 或 ID列表 (`List[int]`)。

```python
# 假设通过 search_envs 得到了ID为 123
env_id_to_disable = 123
await ql_api.disable_envs(env_id_to_disable)
print(f"ID为 {env_id_to_disable} 的环境变量已被禁用。")

await ql_api.EnableEnvs([124, 125])
print("ID为 124 和 125 的环境变量已被启用。")
```

##### **`search_task(keyword)`**
根据关键词搜索定时任务。
- `keyword`: 搜索的关键词。
- **返回**: 包含任务信息的字典。

##### **`run_crons_task(id)`**
运行一个或多个定时任务。
- `id`: 单个任务ID (`str`) 或 ID列表 (`List[str]`)。

```python
tasks_result = await ql_api.search_task("京东签到")
if tasks_result and tasks_result.get('data'):
    task_id = tasks_result['data'][0]['id']
    await ql_api.run_crons_task(task_id)
    print(f"任务 '京东签到' (ID: {task_id}) 已触发运行。")
```

---
#### **`openApiMethod` (底层接口 - 进阶使用)**

此类提供了对青龙 OpenAPI 端点的直接 `GET/PUT/POST/DELETE` 访问，适合需要高度自定义请求的场景。

##### **`get_openApi(query, params=None)`**
执行 `GET` 请求。
- `query`: API路径，如 `'envs'`, `'crons'`, `'configs/config.sh'`。
- `params`: (可选) URL查询参数字典，如 `{'searchValue': 'JD_COOKIE'}`。

```python
# 获取所有环境变量的原始数据
all_envs = await ql_api.get_openApi('envs')
# 获取cron任务列表的第一页（5个）
crons_page1 = await ql_api.get_openApi('crons', params={'page': 1, 'size': 5})
```

##### **`put_openApi(query, json)`**
执行 `PUT` 请求，通常用于更新或执行动作。
- `query`: API路径，如 `'envs/enable'`, `'crons/run'`, `'envs'`。
- `json`: 发送的JSON body。

```python
# 启用ID为 100 和 101 的环境变量
await ql_api.put_openApi('envs/enable', [100, 101])

# 更新ID为 102 的环境变量
await ql_api.put_openApi('envs', {"id": 102, "name": "VAR_NAME", "value": "new_val"})
```

##### **`post_openApi(query, json)`**
执行 `POST` 请求，通常用于创建新资源。
- `query`: API路径，如 `'envs'`, `'configs/save'`。
- `json`: 发送的JSON body。

```python
# 新增一个环境变量
new_env_data = [{"name": "API_KEY", "value": "xyz-123", "remarks": "API密钥"}]
await ql_api.post_openApi('envs', new_env_data)

# 保存配置文件
config_content = "export MY_VAR='hello'"
await ql_api.post_openApi('configs/save', {"name": "config.sh", "content": config_content})
```

##### **`delete_openApi(query, json)`**
执行 `DELETE` 请求，用于删除资源。
- `query`: API路径，如 `'envs'`。
- `json`: 包含要删除资源ID的列表。

```python
# 删除ID为 200 的环境变量
await ql_api.delete_openApi('envs', [200])
```

### **2. 日志工具 (`logging_utils.py`)**

`logging_utils.py` 提供了一个强大且灵活的日志记录工具 `PrintMethodClass`。

#### **核心功能**

- **双重输出**: 日志同时打印到控制台和独立的日志文件 (`./log/脚本名/时间戳.log`)。
- **动态日志格式**: 可以随时向日志前缀中添加或移除上下文信息。
- **环境配置等级**: 通过环境变量 `log_level` 控制日志输出的详细程度。

#### **使用方法**

```python
import asyncio
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("MyAwesomeScript")

async def main():
    log.info("脚本开始运行...")
    log.warning("检测到一个潜在问题。")
    
    # 动态添加上下文信息
    log.set("UserID", "user-001")
    log.info("开始处理用户...")
    log.reset() # 重置上下文
    
    log.error("发生严重错误！", exit=False) # exit=False 表示不退出程序
    log.info("脚本运行结束。")

asyncio.run(main())
```

---
### **3. 异步并发控制器 (`concurrency_utils.py`)**

`RunMethod` 类用于以指定的并发数批量执行异步任务。

#### `RunMethod.ReqConcRun`
最常用的并发运行器。

```python
import asyncio
from utils.concurrency_utils import RunMethod, ReqConcParam
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ConcurrencyDemo")

async def worker_task(item: dict):
    log.info(f"任务 {item['id']} 开始...")
    await asyncio.sleep(item['delay'])
    log.info(f"任务 {item['id']} 完成！")
    return {"id": item['id'], "status": "ok"}

async def main():
    tasks_to_run = [
        {'id': 1, 'delay': 2}, {'id': 2, 'delay': 1}, {'id': 3, 'delay': 3},
        {'id': 4, 'delay': 1}, {'id': 5, 'delay': 2}, {'id': 6, 'delay': 1.5},
    ]

    conc_params = ReqConcParam(func=worker_task, task=tasks_to_run, thread=3, wait=2)

    async for batch_result in RunMethod.ReqConcRun(conc_params):
        log.info(f"--- 一批任务执行完毕 ---")
        for res in batch_result:
            log.info(f"结果: {res.result}")

asyncio.run(main())
```
---
### **4. 异步进程管理器 (`script_executor.py`)**

`ProcessManager` 类能够以非阻塞的方式启动、监控和管理外部脚本或系统命令。

```python
import asyncio
from utils.script_executor import ProcessManager

async def main():
    manager = ProcessManager()
    pid = await manager.run_command(['echo', 'Hello from subprocess!'])
    result_info = await manager.wait_for_process(pid)
    print(f"命令输出:\n{result_info.stdout.strip()}")

asyncio.run(main())
```

---
### **5. 异步安全文件读写 (`async_file_utils.py`)**

`FileMethod` 类提供了一系列带锁的异步文件操作方法。

```python
import asyncio
from utils.async_file_utils import FileMethod

async def main():
    file_path = "my_data.json"
    my_dict = {"name": "Test", "version": 1}
    await FileMethod.write_json(file_path, my_dict, newBuild=True)
    read_data = await FileMethod.read_json(file_path)
    print(f"读取到的JSON: {read_data}")

asyncio.run(main())
```

---
### **6. 环境配置读取 (`env_utils.py`)**

`EnvMethod` 类是读取项目配置的核心。

```python
from utils.env_utils import EnvMethod
# 假设 config.sh 中有: export INT_VAR="123"
int_val = EnvMethod.readEnv("INT_VAR", 0, codeInt=True)
print(f"读取到的整数: {int_val}")
```

---
### **7. HTTP 客户端 (`http_client.py`)**

`AsyncRequestManager` 是一个功能强大的异步HTTP请求器。

```python
import asyncio
from utils.http_client import AsyncRequestManager

async def main():
    req = AsyncRequestManager()
    params = {"method": "GET", "url": "[https://api.ipify.org?format=json](https://api.ipify.org?format=json)"}
    response = await req.async_curl_requests(params, "GetMyIP")
    print(f"我的IP是: {response.text}")

asyncio.run(main())
```

---
### **8. 任务调度器 (`time_scheduler.py`)**

`ServerTimeScheduler` 可用于需要精确时间的任务场景。

```python
import asyncio
from datetime import datetime
from utils.time_scheduler import ServerTimeScheduler
from utils.jd_api import JdApiClient

jd_client = JdApiClient()
time_sync_func = jd_client.Jd_Time

async def main():
    scheduler = ServerTimeScheduler(func=time_sync_func)
    print("等待下一个整点分钟的到来...")
    now = datetime.fromtimestamp(await scheduler.sync_time() / 1000)
    next_minute = (now + timedelta(minutes=1)).replace(second=0, microsecond=0)
    await scheduler.wait_until(hour=next_minute.hour, minute=next_minute.minute)
    print(f"精确时间到达: {datetime.now()}")

asyncio.run(main())
```

---
### **9. User-Agent 生成器 (`user_agent_generator.py`)**

轻松生成各种真实的移动设备User-Agent。

```python
from utils.user_agent_generator import PhoneModel
random_phone = PhoneModel.get_phone_models(brand="Xiaomi")
print(f"随机小米设备: {random_phone.name} ({random_phone.model_number})")
```

---
### **10. 通知发送 (`sendNotify.py`)**

`SendMethod` 负责加载所有启用的推送插件并发送消息。

```python
import asyncio
from datetime import datetime
from utils.sendNotify import SendMethod, SendParam

async def main():
    notify = SendMethod()
    title = "任务报告"
    content = f"所有任务已于 {datetime.now()} 完成。"
    params = SendParam(title=title, content=content)
    await notify.send_all(params)
    print("通知已发送。")

asyncio.run(main())
```

---
### **11. 开发者: 如何添加新的推送插件**

本工具库的通知系统是插件化的，添加新的推送方式非常简单。

1.  在 `function/push_plugins/` 目录下创建一个新的Python文件，例如 `my_pusher.py`。
2.  在该文件中，创建一个继承自 `BaseSender` 的类。
3.  实现 `is_enabled` 和 `send` 两个抽象方法。

**示例 (`function/push_plugins/my_pusher.py`):**

```python
from utils.sendNotify import BaseSender
from utils.env_utils import EnvMethod
import asyncio

class MyPusherSender(BaseSender):
    def __init__(self):
        super().__init__()
        self.is_open = EnvMethod.readEnv("MY_PUSHER_ISOPEN", "false").lower() == "true"
        self.api_key = EnvMethod.readEnv("MY_PUSHER_APIKEY")

    def is_enabled(self) -> bool:
        return self.is_open and bool(self.api_key)

    async def send(self, title: str, content: str, **kwargs) -> bool:
        if not self.is_enabled():
            return False
        # ... 实现你的发送逻辑 ...
        print(f"[MyPusher] 正在发送: {title}")
        return True
```

4.  在 `env/config.sh` 中添加对应的环境变量 (`MY_PUSHER_ISOPEN`, `MY_PUSHER_APIKEY`)。
5.  完成！`SendMethod` 会自动加载并使用你的新插件。

---

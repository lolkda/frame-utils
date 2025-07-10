# Python 高性能异步自动化工具库 (终极版文档)

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/)

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

# --- 推送通知插件配置 (以PushDeer为例) ---
export PUSHDEER_ISOPEN="true"
export PUSHDEER_KEY="PDU123456789xxxx"

# --- 日志等级配置 ---
# 10=DEBUG, 20=INFO, 30=WARNING, 40=ERROR
export log_level="20"
```

---

## 模块使用指南 🛠️

为了便于理解，我们将工具库划分为三个主要部分：

- **一、核心基础**: 编写任何脚本都离不开的基础模块。
- **二、异步与流程控制**: 用于管理复杂异步流程和外部进程的工具。
- **三、辅助与扩展**: 提供通知、文件操作、UA生成等辅助功能。

### 一、 核心基础 (Core Foundations)

这部分是构建自动化任务的基石。

---
#### **1. 日志工具 (`logging_utils.py`)**

`logging_utils.py` 提供了一个强大且灵活的日志记录工具 `PrintMethodClass`，专为自动化脚本设计。它不仅能将日志同时输出到控制台和文件中，还支持动态添加上下文信息，使日志追踪和调试变得异常简单。

##### **核心功能**

- **双重输出**: 日志同时打印到控制台和独立的日志文件 (`./log/脚本名/时间戳.log`)。
- **动态日志格式**: 可以随时向日志前缀中添加或移除上下文信息（如用户名、当前代理状态等），让日志内容更丰富。
- **环境配置等级**: 通过环境变量 `log_level` 控制日志输出的详细程度 (DEBUG, INFO, WARNING, ERROR)。
- **自动错误退出**: 在记录 ERROR 级别日志后，可选择自动终止脚本，防止程序在错误状态下继续运行。
- **自动上下文重置**: 可配置在每次打印日志后自动清除临时添加的上下文信息，保持日志清爽。

##### **测试Demo**
```python
# logging_demo.py
import asyncio
from utils.logging_utils import PrintMethodClass

# 1. 初始化日志记录器，可以传入一个默认的脚本名
log = PrintMethodClass("UserProcessor", auto_reset=False)

async def process_user(user_id, use_proxy):
    # 2. 为当前任务设置上下文
    log.set("UserID", user_id)
    if use_proxy:
        log.set("Network", "Proxy")
    
    log.info("开始处理该用户...")
    await asyncio.sleep(0.5) 
    log.info("用户处理完毕。")

    # 3. 手动清除上下文
    log.reset()

async def main():
    log.info("脚本开始运行...")
    log.warning("检测到一个潜在问题，但不影响运行。")
    log.debug("这是一条非常详细的调试信息。") 
    
    # 4. 演示上下文管理
    await process_user("user-001", use_proxy=False)
    log.info("------")
    await process_user("user-007", use_proxy=True)
    
    # 5. 演示错误处理
    try:
        raise ValueError("示例错误")
    except ValueError as e:
        log.error(f"捕获到一个错误: {e}", exit=False)
    
    log.info("脚本运行结束。")

if __name__ == "__main__":
    asyncio.run(main())
```

---
#### **2. 环境配置读取 (`env_utils.py`)**

`EnvMethod` 类是读取项目配置的核心，提供了比 `os.getenv` 更强大的功能。

##### **核心方法**
- **`readEnv(key, default=None, codeInt=False, codeList=False)`**:
  智能读取环境变量，并能根据需要进行类型转换。
- **`checkEnv(obj_self, config, err_exit=True)`**:
  根据配置清单，结构化地加载环境变量到对象的属性中。
- **`load_config_from_env(obj_self, config_obj, err_exit=True)`**:
  更优雅的配置加载方式，通过类型提示和映射表自动加载。

##### **测试Demo**
```python
# env_demo.py
import os
from typing import List
from utils.env_utils import EnvMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("EnvDemo")

# 模拟设置环境变量
os.environ['DEMO_STR'] = "hello world"
os.environ['DEMO_INT'] = "123"
os.environ['DEMO_LIST'] = "a|b|c"
os.environ['DEMO_JSON_LIST'] = '[1, 2, 3]'
os.environ['DEMO_BOOL'] = 'true'

def demo_read_env():
    log.info("--- 测试 readEnv ---")
    str_val = EnvMethod.readEnv("DEMO_STR", "default")
    int_val = EnvMethod.readEnv("DEMO_INT", 0, codeInt=True)
    list_val = EnvMethod.readEnv("DEMO_LIST", [])
    json_list_val = EnvMethod.readEnv("DEMO_JSON_LIST", [])
    bool_val = EnvMethod.readEnv("DEMO_BOOL", False)
    
    log.info(f"字符串: {str_val} (类型: {type(str_val)})")
    log.info(f"整数: {int_val} (类型: {type(int_val)})")
    log.info(f"列表: {list_val} (类型: {type(list_val)})")
    log.info(f"JSON列表: {json_list_val} (类型: {type(json_list_val)})")
    log.info(f"布尔值: {bool_val} (类型: {type(bool_val)})")

def demo_load_config():
    log.info("\n--- 测试 load_config_from_env ---")
    class AppConfig:
        env_map = {
            'str_setting': 'DEMO_STR',
            'int_setting': 'DEMO_INT',
            'list_setting': 'DEMO_LIST'
        }
        str_setting: str = "default"
        int_setting: int = 0
        list_setting: List[str] = []

    class MyApp:
        def __init__(self):
            EnvMethod.load_config_from_env(self, AppConfig())
    
    app = MyApp()
    log.info(f"App str_setting: {app.str_setting}")
    log.info(f"App int_setting: {app.int_setting}")
    log.info(f"App list_setting: {app.list_setting}")
    
demo_read_env()
demo_load_config()
```
---
#### **3. HTTP 客户端 (`http_client.py`)**

`AsyncRequestManager` 是一个功能强大的异步HTTP请求器，是所有网络请求的基础。

##### `async_curl_requests`
这是发送所有请求的核心方法，通过一个字典来配置请求。

- **`param` 字典常用键**:
  - `method`: `GET`, `POST`, `PUT`, `DELETE` 等。
  - `url`: 请求的URL。
  - `params`: (可选) `GET`请求的查询参数字典。
  - `json`: (可选) `POST`请求的JSON body。
  - `data`: (可选) `POST`请求的表单数据。
  - `headers`: (可选) 请求头字典。
  - `cookies`: (可选) Cookie字典。
  - `proxy`: (可选) `True` 或代理地址字符串，`True` 表示使用配置的代理池。
  - `proxy_retry`: (可选) `True` 表示在请求失败时自动切换成代理重试。

##### **测试Demo**
```python
# http_demo.py
import asyncio
from utils.http_client import AsyncRequestManager
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("HttpDemo")

async def main():
    req = AsyncRequestManager()

    # 1. GET 请求
    get_params = {"method": "GET", "url": "[https://httpbin.org/get?a=1](https://httpbin.org/get?a=1)", "proxy": False}
    res_get = await req.async_curl_requests(get_params, "TestGET")
    log.info(f"GET请求状态码: {res_get.status}")

    # 2. POST 请求
    post_params = {
        "method": "POST",
        "url": "[https://httpbin.org/post](https://httpbin.org/post)",
        "json": {"user": "test", "id": 123},
        "headers": {"X-Custom-Header": "MyValue"}
    }
    res_post = await req.async_curl_requests(post_params, "TestPOST")
    if res_post.status == 200:
        log.info(f"POST请求成功，返回的JSON中包含我们的Header: {'X-Custom-Header' in res_post.text}")
        
if __name__ == "__main__":
    asyncio.run(main())
```

---
#### **4. 青龙面板 API (`openApi.py`)**

`openApi.py` 模块提供了与青龙面板进行交互的全部能力，核心是 `openApiCommonMethod` (推荐) 和 `openApiMethod` (底层) 两个类。

##### **`openApiCommonMethod` (高级封装 - 推荐)**

此类提供了最常用、最便捷的方法，隐藏了底层的API细节，是日常使用的首选。

- **`get_cookie(ContainerName, name)`**: 快速获取指定面板中某个环境变量的所有值。
- **`search_envs(keyword)`**: 根据关键词模糊搜索环境变量。
- **`update_envs(...)`**: 更新或新增一个环境变量。
- **`disable_envs(id) / EnableEnvs(id)`**: 禁用或启用一个或多个环境变量。
- **`search_task(keyword)`**: 根据关键词搜索定时任务。
- **`run_crons_task(id)`**: 运行一个或多个定时任务。

##### **实战测试 Demo**

```python
# openapi_demo.py
import asyncio
from utils.openApi import openApiCommonMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("QL_Demo")

async def main():
    ql_api = openApiCommonMethod()
    ql_api.openApi_Use = "JD"
    var_name = "MY_API_TEST_VAR"
    
    log.info(f"--- 1. 新增/更新环境变量 '{var_name}' ---")
    await ql_api.update_envs(
        name=var_name, value="initial_value_123", remarks="这是一个API测试变量",
        add_env=True, keyword=var_name
    )
    log.info(f"变量 '{var_name}' 已创建或更新。")

    log.info(f"\n--- 2. 搜索并确认变量 ---")
    envs = await ql_api.search_envs(var_name)
    env_id = envs[0]['id']
    log.info(f"找到变量，ID: {env_id}")

    log.info(f"\n--- 3. 禁用与启用 ---")
    await ql_api.disable_envs(env_id)
    log.info(f"变量 ID:{env_id} 已禁用。")
    await ql_api.EnableEnvs(env_id)
    log.info(f"变量 ID:{env_id} 已重新启用。")
    
    log.info(f"\n--- 4. 清理测试变量 ---")
    await ql_api.delete_openApi('envs', [env_id])
    log.info(f"测试变量 ID:{env_id} 已被删除。")

if __name__ == "__main__":
    asyncio.run(main())
```

<br>

### 二、 异步与流程控制 (Async & Flow Control)

这部分工具用于管理复杂的异步流程和外部进程。

---
#### **1. 异步并发控制器 (`concurrency_utils.py`)**

`RunMethod` 类用于以指定的并发数批量执行异步任务。

- **`ReqConcParam(func, task, thread, wait, **kwargs)`**:
  - `func`: 要并发执行的异步函数。
  - `task`: 一个可迭代对象，其中每个元素都将作为 `func` 的第一个参数。
  - `thread`: 最大并发数。
  - `wait`: (可选) 每批任务执行完毕后的等待秒数。
  - `**kwargs`: (可选) 其他需要传递给 `func` 的固定参数。
- **`ReqConcResult`**: 迭代 `ReqConcRun` 返回的结果，包含 `.result` (函数返回值) 和 `.task` (原始任务项)。

##### **测试Demo**
```python
# concurrency_demo.py
import asyncio
from utils.concurrency_utils import RunMethod, ReqConcParam
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ConcurrencyDemo")

async def worker_task(item: dict, extra_param: str):
    log.info(f"任务 {item['id']} ({extra_param}) 开始...")
    await asyncio.sleep(item['delay'])
    log.info(f"任务 {item['id']} 完成！")
    return {"id": item['id'], "status": "ok"}

async def main():
    tasks = [{'id': i, 'delay': round(0.5 + i/5, 1)} for i in range(1, 9)]
    params = ReqConcParam(func=worker_task, task=tasks, thread=3, wait=2, extra_param="Fixed")

    async for batch in RunMethod.ReqConcRun(params):
        log.info("--- 一批任务完成 ---")
        for res in batch:
            log.info(f"  结果: {res.result}, 原始任务: {res.task}")

if __name__ == "__main__":
    asyncio.run(main())
```
---
#### **2. 任务调度器 (`time_scheduler.py`)**

`ServerTimeScheduler` 可用于需要精确时间的任务场景，如整点秒杀。

- **`ServerTimeScheduler(func)`**:
  初始化调度器，`func` 是一个用于获取服务器时间（13位毫秒时间戳）的异步函数。
- **`wait_until(hour, minute, second)`**:
  异步等待直到下一个指定的时:分:秒。
- **`sleep_until_next_active_period(active_hours)`**:
  如果当前不在活跃时间段内，则休眠直到下一个活跃时段开始。`active_hours` 格式为 `["2-5", "8"]`。

##### **测试Demo**
```python
# scheduler_demo.py
import asyncio
from datetime import datetime, timedelta
from utils.time_scheduler import ServerTimeScheduler
from utils.jd_api import JdApiClient
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("SchedulerDemo")
jd_client = JdApiClient()

async def my_timed_task():
    log.info(f"精确时间任务在 {datetime.now()} 执行!")

async def main():
    scheduler = ServerTimeScheduler(func=jd_client.Jd_Time)
    
    target_time = datetime.now() + timedelta(seconds=5)
    log.info(f"当前时间: {datetime.now()}, 准备等待到 {target_time.strftime('%H:%M:%S')}")
    await scheduler.wait_until(
        hour=target_time.hour, minute=target_time.minute, second=target_time.second
    )
    await my_timed_task()

if __name__ == "__main__":
    asyncio.run(main())
```
---
#### **3. 异步进程管理器 (`script_executor.py`)**

`ProcessManager` 类能够以非阻塞的方式启动、监控和管理外部脚本或系统命令。

- **`run_script(script_path, run_method='auto', args=[], callback=None)`**:
  异步执行一个脚本文件。
- **`run_command(command, callback=None)`**:
  异步执行一个系统命令。
- **`wait_for_process(pid, timeout=None)`**:
  等待指定`pid`的进程结束并返回结果。
- **`kill_process(pid)`**:
  终止一个正在运行的进程。

##### **测试Demo**
```python
# executor_demo.py
import asyncio
from utils.script_executor import ProcessManager, ProcessInfo
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ExecutorDemo")

with open("my_test_script.py", "w") as f:
    f.write('import time, sys\nprint(f"脚本运行中，参数: {sys.argv[1:]}")\ntime.sleep(2)\nprint("脚本结束")')

def my_callback(info: ProcessInfo):
    log.info(f"--- 回调触发 (PID: {info.pid}) ---")
    log.info(f"状态: {info.status}, 返回码: {info.return_code}")

async def main():
    manager = ProcessManager()
    
    log.info("准备运行 Python 脚本...")
    pid = await manager.run_script('my_test_script.py', args=['arg1'], callback=my_callback)
    log.info(f"脚本已启动，PID: {pid}")
    
    result_info = await manager.wait_for_process(pid)
    
    log.info("\n--- 主程序获取到最终结果 ---")
    log.info(f"脚本输出:\n{result_info.stdout.strip()}")

if __name__ == "__main__":
    asyncio.run(main())
```

<br>

### 三、 辅助与扩展 (Utilities & Extensions)

这部分模块提供了通知、文件I/O、UA生成等实用功能。

---
#### **1. 通知发送 (`sendNotify.py`)**

`sendNotify.py` 模块提供了一个可扩展的通知发送框架。

##### **核心用法**
`SendMethod` 类会自动扫描 `function/push_plugins/` 目录下的所有插件，并加载已启用的插件。

- **`SendParam(title, content, uids=None)`**: 用于封装通知内容的标准数据类。
- **`send_all(param)`**: 向所有启用的渠道广播消息。
- **`send_to(sender_class, param)`**: 向指定的单个渠道发送消息。

##### **测试Demo**
```python
# notify_demo.py
import asyncio
from datetime import datetime
from utils.sendNotify import SendMethod, SendParam
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("NotifyDemo")

async def main():
    notify = SendMethod()
    
    if not notify.senders:
        log.warning("未配置或启用任何推送器，无法发送通知。")
        return

    params = SendParam(
        title="自动化任务报告", 
        content=f"所有任务已于 {datetime.now()} 完成。"
    )
    
    log.info("准备向所有启用的渠道发送通知...")
    await notify.send_all(params)

if __name__ == "__main__":
    asyncio.run(main())
```
---
##### **开发者指南：如何添加新的推送插件 (保姆级教程)**

本工具库的通知系统是插件化的，这意味您可以非常轻松地添加任何您想使用的推送服务。下面我们以添加一个虚构的 **PushDeer** 推送服务为例，一步步教您如何操作。

###### **第一步: 创建插件文件**
在 `function/push_plugins/` 目录下，创建一个新的Python文件，命名为您推送服务的名字，例如 `pushdeer.py`。

###### **第二步: 编写类结构**
打开 `pushdeer.py` 文件，写入以下基础代码。我们创建一个 `PushDeerSender` 类，并让它继承自 `BaseSender`。

```python
# function/push_plugins/pushdeer.py
from utils.sendNotify import BaseSender
from utils.env_utils import EnvMethod
import asyncio

class PushDeerSender(BaseSender):
    """
    PushDeer 推送插件
    """
    def __init__(self):
        super().__init__()
        self.is_open = EnvMethod.readEnv("PUSHDEER_ISOPEN", "false").lower() == "true"
        self.key = EnvMethod.readEnv("PUSHDEER_KEY")
        self.api_url = "[https://api2.pushdeer.com/message/push](https://api2.pushdeer.com/message/push)"

    def is_enabled(self) -> bool:
        return self.is_open and bool(self.key)

    async def send(self, title: str, content: str, **kwargs) -> bool:
        if not self.is_enabled():
            return False
            
        params = {
            "method": "POST",
            "url": self.api_url,
            "data": {"pushkey": self.key, "text": title, "desp": content}
        }
        
        response = await self.req.async_curl_requests(params, "PushDeer")
        
        if response.status == 200 and response.json().get("code") == 0:
            self.log.info("PushDeer 推送成功！")
            return True
        
        self.log.error(f"PushDeer 推送失败: {response.text}")
        return False
```

###### **第三步: 添加配置到 `config.sh`**
打开 `/env/config.sh` 文件，在末尾添加PushDeer的配置。

```shell
# --- 推送通知插件配置 (PushDeer示例) ---
export PUSHDEER_ISOPEN="true"
export PUSHDEER_KEY="PDU123456789xxxx" # 替换成你自己的PushDeer Key
```

###### **完成!**
至此，您已成功添加了一个全新的推送插件。当您在主脚本中调用 `SendMethod().send_all(...)` 时，程序会自动发现并执行您的 `PushDeerSender`。

---
#### **2. 异步安全文件读写 (`async_file_utils.py`)**

`FileMethod` 类提供了一系列带锁的异步文件操作方法。

##### **测试Demo**
```python
# file_demo.py
import asyncio
import os
from utils.async_file_utils import FileMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("FileDemo")
JSON_FILE = "demo_data.json"

async def main():
    await FileMethod.write_json(JSON_FILE, {"count": 1}, newBuild=True)
    log.info(f"已创建JSON文件: {await FileMethod.read_str(JSON_FILE)}")
    
    await FileMethod.write_json(JSON_FILE, {"status": "active"})
    log.info(f"更新后: {await FileMethod.read_str(JSON_FILE)}")

    if os.path.exists(JSON_FILE): os.remove(JSON_FILE)

if __name__ == "__main__":
    asyncio.run(main())
```
---
#### **3. User-Agent 生成器 (`user_agent_generator.py`)**

轻松生成各种真实的移动设备User-Agent。

- `PhoneModel.get_phone_models(brand=None)`: 获取一个随机设备信息。
- `JdUserAgentGenerator(clientVersion, build)`: 专门生成京东App的UA。

##### **测试Demo**
```python
# ua_demo.py
from utils.user_agent_generator import PhoneModel
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("UADemo")
random_phone = PhoneModel.get_phone_models(brand="Xiaomi")
log.info(f"随机小米设备: {random_phone.name} ({random_phone.model_number})")
```

---

## 许可证

本项目采用 [MIT](https://opensource.org/licenses/MIT) 许可证。

# Python 高性能异步自动化工具库

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

### **1. 日志工具 (`logging_utils.py`)**

`logging_utils.py` 提供了一个强大且灵活的日志记录工具 `PrintMethodClass`，专为自动化脚本设计。它不仅能将日志同时输出到控制台和文件中，还支持动态添加上下文信息，使日志追踪和调试变得异常简单。

#### **核心功能**

- **双重输出**: 日志同时打印到控制台和独立的日志文件 (`./log/脚本名/时间戳.log`)。
- **动态日志格式**: 可以随时向日志前缀中添加或移除上下文信息（如用户名、当前代理状态等），让日志内容更丰富。
- **环境配置等级**: 通过环境变量 `log_level` 控制日志输出的详细程度 (DEBUG, INFO, WARNING, ERROR)。
- **自动错误退出**: 在记录 ERROR 级别日志后，可选择自动终止脚本，防止程序在错误状态下继续运行。
- **自动上下文重置**: 可配置在每次打印日志后自动清除临时添加的上下文信息，保持日志清爽。

#### **使用方法**

##### **基本初始化和日志记录**

最简单的使用方式是直接实例化 `PrintMethodClass` 并调用其方法。

- `info(content)`: 记录普通信息。
- `warning(content)`: 记录警告。
- `debug(content)`: 记录用于调试的详细信息（仅当 `log_level` <= 10 时显示）。
- `error(content, exit=True)`: 记录错误。`exit` 参数默认为 `True`，会在记录后终止脚本。

```python
import asyncio
from utils.logging_utils import PrintMethodClass

# 1. 初始化日志记录器，可以传入一个默认的脚本名
log = PrintMethodClass("MyAwesomeScript")

async def main():
    log.info("脚本开始运行...")
    log.warning("检测到一个潜在问题，但不影响运行。")
    log.debug("这是一条非常详细的调试信息。") # 默认不会显示
    
    try:
        # 模拟一个可能出错的操作
        raise ValueError("示例错误")
    except ValueError as e:
        # 记录错误，但程序不会退出，因为 exit=False
        log.error(f"捕获到一个错误: {e}", exit=False)
    
    log.info("脚本运行结束。")

# 运行
asyncio.run(main())
```
**输出示例:**
```
18:45:01 | INFO     | [MyAwesomeScript] : 脚本开始运行...
18:45:01 | WARNING  | [MyAwesomeScript] : 检测到一个潜在问题，但不影响运行。
18:45:01 | ERROR    | [MyAwesomeScript] : 捕获到一个错误: 示例错误
18:45:01 | INFO     | [MyAwesomeScript] : 脚本运行结束。
```

##### **动态上下文管理 (核心功能)**

这是 `PrintMethodClass` 最强大的功能。您可以使用 `set`, `remove`, 和 `reset` 方法来动态修改单条日志的上下文信息。

- `set(key, value)`: 添加或修改一个上下文信息。
- `remove(key)`: 移除一个指定的上下文信息。
- `reset()`: 清除所有临时的上下文信息（不包括初始化时设置的脚本名）。

```python
import asyncio
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("UserProcessor")

async def process_user(user_id, use_proxy):
    # 为当前任务设置上下文
    log.set("UserID", user_id)
    if use_proxy:
        log.set("Network", "Proxy")

    log.info("开始处理该用户...")
    await asyncio.sleep(1) # 模拟操作
    log.info("用户处理完毕。")

    # 默认情况下，每次调用log方法后，上下文会自动重置。
    # 如果初始化时设置了 auto_reset=False，则需要手动调用 log.reset()

async def main():
    await process_user("user-001", use_proxy=False)
    # 此时上下文已自动重置
    log.info("------")
    await process_user("user-007", use_proxy=True)

# 运行
asyncio.run(main())
```

**输出示例:**
```
18:45:05 | INFO     | [UserProcessor] | [UserID: user-001] : 开始处理该用户...
18:45:06 | INFO     | [UserProcessor] | [UserID: user-001] : 用户处理完毕。
18:45:06 | INFO     | [UserProcessor] : ------
18:45:06 | INFO     | [UserProcessor] | [UserID: user-007] | [Network: Proxy] : 开始处理该用户...
18:45:07 | INFO     | [UserProcessor] | [UserID: user-007] | [Network: Proxy] : 用户处理完毕。
```

##### **控制上下文的自动重置**

在初始化 `PrintMethodClass` 时，可以通过 `auto_reset=False` 参数控制上下文是否在每次打印后自动清除。

```python
import asyncio
from utils.logging_utils import PrintMethodClass

# auto_reset=False 表示上下文会一直保留，直到手动清除
log = PrintMethodClass("DataBatch", auto_reset=False)

async def main():
    log.set("BatchID", "batch-9527")
    
    log.info("开始处理批次...") # 输出: [DataBatch] | [BatchID: batch-9527] : 开始处理批次...
    
    # 因为 auto_reset=False，所以 BatchID 这个上下文信息会保留
    log.info("正在处理项目A...") # 输出: [DataBatch] | [BatchID: batch-9527] : 正在处理项目A...
    
    log.set("ItemID", "item-01")
    log.info("处理细节...") # 输出: [DataBatch] | [BatchID: batch-9527] | [ItemID: item-01] : 处理细节...
    
    log.remove("ItemID") # 手动移除一个上下文
    log.info("项目A处理完毕。") # 输出: [DataBatch] | [BatchID: batch-9527] : 项目A处理完毕。
    
    # 手动重置所有临时上下文
    log.reset()
    log.info("批次处理完成。") # 输出: [DataBatch] : 批次处理完成。
    
asyncio.run(main())
```

### **2. 异步并发控制器 (`concurrency_utils.py`)**

`concurrency_utils` 模块提供了强大的工具来管理和执行大量异步任务，核心是 `RunMethod` 类。

#### `RunMethod.ReqConcRun`
这是最常用、最灵活的并发运行器。它接收一个 `ReqConcParam` 对象，按指定的并发数分批执行任务，并可在每批任务间设置等待时间。

- **`ReqConcParam(func, task, thread, wait, **kwargs)`**:
  - `func`: 要并发执行的异步函数。
  - `task`: 一个可迭代对象（如列表），其中每个元素都将作为 `func` 的第一个参数。
  - `thread`: 最大并发数。
  - `wait`: (可选) 每批任务执行完毕后的等待秒数。
  - `**kwargs`: (可选) 其他需要传递给 `func` 的固定参数。
- **`ReqConcResult`**: 迭代 `ReqConcRun` 返回的结果，每个元素都是此对象，包含 `.result` (函数的返回值) 和 `.task` (原始任务项)。

```python
import asyncio
import random
from utils.concurrency_utils import RunMethod, ReqConcParam
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ConcurrencyDemo")

# 模拟一个耗时的异步任务，例如API请求
async def worker_task(item: dict, extra_param: str):
    task_id = item['id']
    delay = item['delay']
    log.info(f"任务 {task_id} ({extra_param}) 开始，预计耗时 {delay} 秒...")
    await asyncio.sleep(delay)
    log.info(f"任务 {task_id} 完成！")
    return {"id": task_id, "status": "ok"}

async def main():
    # 准备一批任务数据
    tasks_to_run = [
        {'id': 1, 'delay': 2}, {'id': 2, 'delay': 1}, {'id': 3, 'delay': 3},
        {'id': 4, 'delay': 1}, {'id': 5, 'delay': 2}, {'id': 6, 'delay': 1.5},
        {'id': 7, 'delay': 2.5}, {'id': 8, 'delay': 1},
    ]

    # 配置并发参数
    conc_params = ReqConcParam(
        func=worker_task,
        task=tasks_to_run,
        thread=3,  # 同时只运行3个任务
        wait=2,     # 每执行完一批（3个）任务后，等待2秒
        extra_param="FixedValue" # 额外的固定参数
    )

    # 迭代运行器，获取每批次的结果
    async for batch_result in RunMethod.ReqConcRun(conc_params):
        log.info(f"--- 一批任务执行完毕 ---")
        for res in batch_result:
            log.info(f"结果: {res.result}, 原始任务: {res.task}")

asyncio.run(main())
```

---
### **3. 异步进程管理器 (`script_executor.py`)**

`ProcessManager` 类能够以非阻塞的方式启动、监控和管理外部脚本或系统命令。

#### `ProcessManager.run_script`
异步执行一个脚本文件（.py, .js, .sh 等）。

- **`run_script(script_path, run_method='auto', args=[], callback=None, **kwargs)`**:
  - `script_path`: 脚本的路径。
  - `run_method`: (可选) 运行方式，`auto` 会根据文件后缀自动检测。
  - `args`: (可选) 传递给脚本的命令行参数列表。
  - `callback`: (可选) 进程结束后要执行的回调函数。

#### `ProcessManager.wait_for_process`
等待一个已启动的进程执行完毕，并获取其结果。

- **`wait_for_process(pid, timeout=None)`**:
  - `pid`: `run_script` 或 `run_command` 返回的进程ID。
  - `timeout`: (可选) 最大等待时间（秒）。
- **返回**: 一个 `ProcessInfo` 对象，包含 `.status`, `.return_code`, `.stdout`, `.stderr` 等信息。

```python
import asyncio
from utils.script_executor import ProcessManager

# 创建一个用于测试的脚本文件
with open("test_script.py", "w") as f:
    f.write('import time, sys\n')
    f.write('print(f"脚本开始，接收到参数: {sys.argv[1:]}")\n')
    f.write('time.sleep(2)\n')
    f.write('print("脚本结束")\n')

def my_callback(info):
    print(f"--- 回调函数触发 (PID: {info.pid}) ---")
    print(f"脚本 '{info.script_path}' 已结束，状态: {info.status}")

async def main():
    manager = ProcessManager()
    
    print("准备运行 test_script.py...")
    
    pid = await manager.run_script(
        'test_script.py', 
        args=['arg1', 'arg2'],
        callback=my_callback
    )
    print(f"脚本已启动，PID: {pid}")
    
    # 你可以在这里做其他事，脚本在后台运行
    print("主程序继续执行其他任务...")
    await asyncio.sleep(1)
    
    # 现在等待脚本执行完成
    print(f"等待进程 {pid} 结束...")
    result_info = await manager.wait_for_process(pid)
    
    print(f"\n--- 主程序获取到结果 ---")
    print(f"状态: {result_info.status}")
    print(f"返回码: {result_info.return_code}")
    print(f"标准输出:\n{result_info.stdout.strip()}")

asyncio.run(main())
```

---
### **4. 异步安全文件读写 (`async_file_utils.py`)**

`FileMethod` 类提供了一系列带锁的异步文件操作方法，确保在并发环境下的文件写入安全。

#### `FileMethod.write_json / read_json`
安全地读写 JSON 文件。`write_json` 支持新建文件、更新字典和追加列表。

- **`write_json(file_name, updata, newBuild=False)`**:
  - `updata`: 要写入的数据。
  - `newBuild=True`: 会用 `updata` 的内容直接覆盖或创建新文件。
  - `newBuild=False`: 会先读取 `file_name`，如果内容是字典则更新，是列表则追加。

```python
import asyncio
from os import path
from utils.async_file_utils import FileMethod

async def main():
    file_path = "my_data.json"
    
    # 示例1: 写入和读取JSON
    my_dict = {"name": "Test", "version": 1, "tags": ["a", "b"]}
    
    # 第一次写入，新建文件
    await FileMethod.write_json(file_path, my_dict, newBuild=True)
    print(f"JSON已写入到 {file_path}")
    
    read_data = await FileMethod.read_json(file_path)
    print(f"读取到的JSON: {read_data}")
    
    # 更新JSON文件 (添加/修改键值)
    update_info = {"version": 2, "author": "Gemini"}
    await FileMethod.write_json(file_path, update_info)
    print("JSON文件已更新。")
    
    updated_data = await FileMethod.read_json(file_path)
    print(f"更新后的JSON: {updated_data}")
    
    # 示例2: 写入和读取文本
    text_file = "log.txt"
    await FileMethod.write_str(text_file, "第一行日志\n", mode="w") # w模式覆盖
    await FileMethod.write_str(text_file, "第二行日志\n", mode="a") # a模式追加
    
    content = await FileMethod.read_str(text_file)
    print(f"\n读取到的文本内容:\n{content}")

# 运行
asyncio.run(main())
```

---

### **5. 环境配置读取 (`env_utils.py`)**

`EnvMethod` 类是读取项目配置的核心，提供了比 `os.getenv` 更强大的功能。

#### `EnvMethod.readEnv`
智能读取环境变量，并能根据需要进行类型转换。

- **`readEnv(key, default=None, codeInt=False, codeList=False)`**:
  - `key`: 环境变量名。
  - `default`: (可选) 变量不存在时返回的默认值。
  - `codeInt=True`: 将结果转换为整数或整数列表。
  - `codeList=True`: 强制将结果按 `|` 分割为列表。

```python
from utils.env_utils import EnvMethod

# 假设 env/config.sh 中有以下内容:
# export STR_VAR="hello"
# export INT_VAR="123"
# export LIST_VAR="item1|item2"
# export BOOL_VAR="true"
# export JSON_LIST_VAR='["a", "b"]'

# 读取字符串
str_val = EnvMethod.readEnv("STR_VAR", "default_str")
print(f"字符串: {str_val}")

# 读取整数
int_val = EnvMethod.readEnv("INT_VAR", 0, codeInt=True)
print(f"整数: {int_val}")

# 读取布尔值
bool_val = EnvMethod.readEnv("BOOL_VAR", False)
print(f"布尔值: {bool_val}")

# 读取管道符分割的列表
list_val = EnvMethod.readEnv("LIST_VAR", [])
print(f"管道符列表: {list_val}")

# 读取JSON格式的列表
json_list_val = EnvMethod.readEnv("JSON_LIST_VAR", [])
print(f"JSON列表: {json_list_val}")
```

---
### **6. HTTP 客户端 (`http_client.py`)**

`AsyncRequestManager` 是一个功能强大的异步HTTP请求器。

#### `async_curl_requests`
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

```python
import asyncio
from utils.http_client import AsyncRequestManager

async def main():
    req = AsyncRequestManager()

    # 示例: 发送一个带自定义头的POST请求
    post_params = {
        "method": "POST",
        "url": "[https://httpbin.org/post](https://httpbin.org/post)",
        "json": {"user": "test", "id": 123},
        "headers": {"X-Custom-Header": "MyValue"},
        "proxy": False # 本次请求不使用代理
    }

    response = await req.async_curl_requests(post_params, "TestPost")
    
    if response.status == 200:
        print("POST请求成功，返回数据:")
        print(response.text)
    else:
        print(f"请求失败，状态码: {response.status}")

asyncio.run(main())
```

---
### **7. 青龙面板 API (`openApi.py`)**

`openApiCommonMethod` 类封装了与青龙面板交互的常用操作。

#### 核心用法
1.  实例化 `openApiCommonMethod`。
2.  设置 `ql_api.openApi_Use = "PREFIX"` 来选择要操作的面板实例（`PREFIX` 对应 `config.sh` 中的 `OPENAPI_PREFIX_...`）。

#### 常用方法
- `get_cookie(ContainerName, name)`: 获取指定面板中某个环境变量的所有值。
- `search_envs(keyword)`: 根据关键词搜索环境变量。
- `update_envs(name, value, remarks, keyword)`: 更新指定的环境变量。
- `search_task(keyword)`: 根据关键词搜索定时任务。
- `run_crons_task(id)`: 运行指定ID的定时任务。

```python
import asyncio
from utils.openApi import openApiCommonMethod

async def main():
    ql_api = openApiCommonMethod()
    
    # 选择要操作的青龙面板
    ql_api.openApi_Use = "JD"
    
    # 示例：获取所有JD_COOKIE
    print("正在获取 JD_COOKIE...")
    cookies = await ql_api.get_cookie("JD", "JD_COOKIE")
    if cookies:
        print(f"成功获取 {len(cookies)} 个Cookie。")
    
    # 示例：运行一个任务
    print("\n正在搜索 '京东签到' 任务...")
    tasks_result = await ql_api.search_task("京东签到")
    if tasks_result and tasks_result.get('data'):
        task_id = tasks_result['data'][0]['id']
        task_name = tasks_result['data'][0]['name']
        print(f"找到任务 '{task_name}', ID: {task_id}。准备运行...")
        await ql_api.run_crons_task(task_id)
        print("任务已触发。")

asyncio.run(main())
```

---
### **8. 任务调度器 (`time_scheduler.py`)**

`ServerTimeScheduler` 可用于需要精确时间的任务场景，如整点秒杀。

#### 核心用法
1.  实例化 `ServerTimeScheduler(func)`，`func` 是一个用于获取服务器时间（13位毫秒时间戳）的异步函数。
2.  使用 `await scheduler.wait_until(...)` 或 `await scheduler.sleep_until_next_active_period(...)` 来等待。

```python
import asyncio
from datetime import datetime
from utils.time_scheduler import ServerTimeScheduler
from utils.jd_api import JdApiClient # 使用京东API获取服务器时间

# 使用京东API作为时间同步源
jd_client = JdApiClient()
time_sync_func = jd_client.Jd_Time

async def搶購任務():
    print(f"抢购任务在精确时间点执行: {datetime.now()}")

async def main():
    scheduler = ServerTimeScheduler(func=time_sync_func)
    
    # 等待到下一个22点整
    print("等待下一个22:00:00...")
    await scheduler.wait_until(hour=22, minute=0, second=0)
    await 搶購任務()
    
    # 等待到下一个活跃时间段
    # "2-5" 表示 2:00 到 5:59, "8" 表示 8:00 到 8:59
    await scheduler.sleep_until_next_active_period(active_hours=["0-2", "8-10"])
    print("到达活跃时段，开始执行日常任务。")

asyncio.run(main())
```

---
### **9. User-Agent 生成器 (`user_agent_generator.py`)**

轻松生成各种真实的移动设备User-Agent。

- `PhoneModel.get_phone_models(brand=None)`: 获取一个随机设备信息，`brand` 参数可选，用于指定品牌。
- `JdUserAgentGenerator(clientVersion, build)`: 专门生成京东App的UA。

```python
from utils.user_agent_generator import PhoneModel, JdUserAgentGenerator

# 示例1: 获取一个随机华为设备信息
random_huawei = PhoneModel.get_phone_models(brand="HUAWEI")
print(f"随机华为设备: {random_huawei.name}")
print(f"型号: {random_huawei.model_number}")
print(f"代码名: {random_huawei.code_name}")

# 示例2: 生成京东App的UA
jd_ua_gen = JdUserAgentGenerator(clientVersion="12.3.4", build="100500")
jd_ua_pair = jd_ua_gen.jd_app_ua(name="jd")
print(f"\n京东App主UA: {jd_ua_pair.app}")
print(f"京东App OkHttp UA: {jd_ua_pair.okhttp}")
```

---
### **10. 通知发送 (`sendNotify.py`)**

`SendMethod` 负责加载所有启用的推送插件并发送消息。

- `SendParam(title, content, uids)`: 用于封装通知内容的标准数据类。`content` 可以是字符串或列表。
- `SendMethod.send_all(param)`: 向所有启用的渠道广播消息。
- `SendMethod.send_to(sender_class, param)`: 向指定的单个渠道发送消息。

```python
import asyncio
from datetime import datetime
from utils.sendNotify import SendMethod, SendParam
# from function.push_plugins.telegram import TelegramSender # 导入具体的插件类

async def main():
    notify = SendMethod()
    
    # 构建通知内容
    title = "自动化任务每日报告"
    content_lines = [
        "✅ 任务A: 成功",
        "❌ 任务B: 失败 - Cookie失效",
        f"🕒 报告时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
    ]
    
    # 创建发送参数
    params = SendParam(title=title, content=content_lines)
    
    # 发送给所有在 config.sh 中启用的通知渠道
    await notify.send_all(params)

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
    """
    我的自定义推送器。
    """
    def __init__(self):
        super().__init__()
        self.is_open = EnvMethod.readEnv("MY_PUSHER_ISOPEN", "false").lower() == "true"
        self.api_key = EnvMethod.readEnv("MY_PUSHER_APIKEY")

    def is_enabled(self) -> bool:
        return self.is_open and bool(self.api_key)

    async def send(self, title: str, content: str, **kwargs) -> bool:
        if not self.is_enabled():
            return False
            
        url = "[https://api.mypusher.com/push](https://api.mypusher.com/push)"
        payload = {"key": self.api_key, "title": title, "message": content}
        params = {"method": "POST", "url": url, "json": payload}
        
        response = await self.req.async_curl_requests(params, "MyPusher")
        
        if response.status == 200:
            self.log.info("我的推送器发送成功！")
            return True
        else:
            self.log.error(f"我的推送器发送失败: {response.text}")
            return False
```

4.  在 `env/config.sh` 中添加对应的环境变量 (`MY_PUSHER_ISOPEN`, `MY_PUSHER_APIKEY`)。
5.  完成！`SendMethod` 会自动加载并使用你的新插件。

---

## 许可证

本项目采用 [MIT](https://opensource.org/licenses/MIT) 许可证。

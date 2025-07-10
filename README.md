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

## 项目结构

```
.
├── utils/
│   ├── push_plugins/       # 推送插件目录
│   ├── jd_api.py           # 封装的京东API
│   ├── env_utils.py        # 环境配置读取工具
│   ├── http_client.py      # 异步HTTP客户端
│   ├── logging_utils.py    # 日志工具
│   ├── openApi.py          # 青龙面板API客户端
│   ├── sendNotify.py       # 推送通知管理器
│   ├── time_scheduler.py   # 异步任务调度器
│   ├── user_agent_generator.py # UA生成器
│   ├── script_executor.py  # 异步进程管理器
│   ├── async_file_utils.py # 异步文件工具
│   └── concurrency_utils.py # 异步并发控制器
├── env/
│   └── config.sh           # 唯一的项目配置文件
└── main_script.py          # 你的主脚本文件
```


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

# --- 推送通知插件配置 (以PushDeer为例) ---
export PUSHDEER_ISOPEN="true"
export PUSHDEER_KEY="PDU123456789xxxx"

# --- 日志等级配置 ---
# 10=DEBUG, 20=INFO, 30=WARNING, 40=ERROR
export log_level="20"
```

---

## 模块使用指南 🛠️

### **1. 异步HTTP客户端 (`http_client.py`)**

## 核心功能 ✨

- **高性能异步请求**: 利用 `curl_cffi` 库，支持HTTP/2和TLS指纹伪装，能更好地模拟真实浏览器行为，有效规避WAF检测。
- **单例模式**: `AsyncRequestManager` 和 `ProxyManager` 均采用单例模式，确保在整个应用中只有一个实例，高效管理连接和资源。
- **智能代理管理**: `ProxyManager` 可自动从配置的API地址获取和轮换代理IP，支持代理池和单个API两种模式。
- **自动重试机制**: `async_curl_requests` 方法内置了失败重试逻辑，提高了脚本的健壮性。
- **灵活的请求配置**: 所有请求参数通过一个字典对象传递，清晰明了，易于构造和修改。

## 核心类与方法详解

### `AsyncRequestManager`

这是您用来发送所有HTTP请求的主类。它是一个单例，您无需担心多次实例化会造成资源浪费。

#### `async def async_curl_requests(self, param: Dict, name: str, retry: int = 10) -> AsyncResponse`
这是发送所有请求的核心方法。

- **参数**:
    - `param` (`Dict`): 一个包含所有请求配置的字典，关键字段如下：
        - `method` (str): **必需**。HTTP方法，如 `'GET'`, `'POST'`, `'PUT'`。
        - `url` (str): **必需**。请求的目标URL。
        - `headers` (Dict, optional): 请求头。
        - `cookies` (Dict, optional): 要附加到请求的Cookie。
        - `params` (Dict, optional): URL的查询参数，用于GET请求。
        - `json` (Dict, optional): 发送JSON格式的请求体。
        - `data` (Dict or str, optional): 发送 `application/x-www-form-urlencoded` 格式的表单数据。
        - `proxy` (bool or str, optional): 是否使用代理。
            - 设置为 `True`: 将从 `ProxyManager` 自动获取一个代理。
            - 设置为代理字符串 (如 `'http://user:pass@host:port'`): 将直接使用该代理。
            - 默认为 `False` 或不设置，不使用代理。
        - `proxy_retry` (bool, optional): `True` 时，如果初始请求失败，会自动切换为使用代理进行重试。
        - `timeout` (int, optional): 本次请求的超时时间（秒），会覆盖全局配置。
        - `allow_redirects` (bool, optional): 是否允许重定向，默认为 `False`。
        - `clear` (bool, optional): `True` 时，请求完成后会清空会话中的Cookie。
    - `name` (`str`): 本次请求的名称，主要用于日志记录，便于问题排查。
    - `retry` (`int`, optional): 最大重试次数，默认为10。

- **返回**:
  - `AsyncResponse`: 一个包含响应信息的数据对象。

### `AsyncResponse`

`async_curl_requests` 方法成功后的返回值类型，包含了所有你需要的回应信息。

- **属性**:
    - `status` (`int`): HTTP状态码，如 `200`, `404`。
    - `headers` (`Dict`): 响应头。
    - `text` (`str`): 响应体文本内容。
    - `url` (`str`): 经过重定向（如果允许）后的最终URL。

### `ProxyManager`

在后台工作的代理管理器，您通常不需要直接与之交互。当您在请求参数中设置 `proxy=True` 时，`AsyncRequestManager` 会自动调用它来获取代理IP。它的行为由 `config.sh` 中的 `PROXY_URL` 和 `PROXY_Method` 环境变量控制。

---

## 实战测试 Demo

下面的Demo将演示如何使用 `AsyncRequestManager` 来执行不同类型的HTTP请求。

```python
# http_client_demo.py
import asyncio
import json
from utils.http_client import AsyncRequestManager
from utils.logging_utils import PrintMethodClass

# 初始化日志记录器，以便看到请求过程中的输出
log = PrintMethodClass("HttpClientDemo")

async def main():
    # 获取AsyncRequestManager的单例实例
    req = AsyncRequestManager()

    log.info("--- 1. 发送一个简单的 GET 请求 ---")
    get_params = {
        "method": "GET",
        "url": "[https://httpbin.org/get](https://httpbin.org/get)",
        "params": {"param1": "value1", "source": "demo"},
        "headers": {"Accept": "application/json"}
    }
    response_get = await req.async_curl_requests(get_params, "TestGET")
    if response_get.status == 200:
        log.info(f"GET 请求成功，状态码: {response_get.status}")
        # 解析返回的JSON并打印'args'字段
        data = json.loads(response_get.text)
        log.info(f"服务器收到的GET参数: {data.get('args')}")
    else:
        log.error(f"GET 请求失败，状态码: {response_get.status}")


    log.info("\n--- 2. 发送一个带 JSON Body 的 POST 请求 ---")
    post_params = {
        "method": "POST",
        "url": "[https://httpbin.org/post](https://httpbin.org/post)",
        "json": {"user": "test_user", "id": 12345},
        "headers": {"X-Request-By": "DemoScript"}
    }
    response_post = await req.async_curl_requests(post_params, "TestPOST")
    if response_post.status == 200:
        log.info(f"POST 请求成功，状态码: {response_post.status}")
        data = json.loads(response_post.text)
        log.info(f"服务器收到的JSON数据: {data.get('json')}")
        log.info(f"服务器收到的自定义请求头: {data.get('headers').get('X-Request-By')}")
    else:
        log.error(f"POST 请求失败，状态码: {response_post.status}")


    log.info("\n--- 3. 演示代理请求 (需配置好 PROXY_URL) ---")
    proxy_params = {
        "method": "GET",
        "url": "[https://api.ipify.org?format=json](https://api.ipify.org?format=json)",
        "proxy": True # 明确指示使用代理
    }
    response_proxy = await req.async_curl_requests(proxy_params, "TestProxyGET")
    if response_proxy.status == 200:
        log.info("代理请求成功！")
        log.info(f"通过代理获取到的IP: {response_proxy.text.strip()}")
    else:
        log.warning(f"代理请求失败或未配置代理，状态码: {response_proxy.status}")


if __name__ == "__main__":
    # 确保您的 config.sh 中有代理配置，否则第三个示例会失败
    # 例如: export PROXY_URL="http://your_proxy_api"
    #      export PROXY_Method="pool"
    asyncio.run(main())

```

### 预期的打印结果

```
[TIME] | INFO     | [HttpClientDemo] : --- 1. 发送一个简单的 GET 请求 ---
[TIME] | INFO     | [HttpClientDemo] | [Local] : GET 请求成功，状态码: 200
[TIME] | INFO     | [HttpClientDemo] | [Local] : 服务器收到的GET参数: {'param1': 'value1', 'source': 'demo'}
[TIME] | INFO     | [HttpClientDemo] : 
--- 2. 发送一个带 JSON Body 的 POST 请求 ---
[TIME] | INFO     | [HttpClientDemo] | [Local] : POST 请求成功，状态码: 200
[TIME] | INFO     | [HttpClientDemo] | [Local] : 服务器收到的JSON数据: {'user': 'test_user', 'id': 12345}
[TIME] | INFO     | [HttpClientDemo] | [Local] : 服务器收到的自定义请求头: DemoScript
[TIME] | INFO     | [HttpClientDemo] : 
--- 3. 演示代理请求 (需配置好 PROXY_URL) ---
[TIME] | INFO     | [HttpClientDemo] | [Proxy] : 代理请求成功！
[TIME] | INFO     | [HttpClientDemo] | [Proxy] : 通过代理获取到的IP: {"ip":"xxx.xxx.xxx.xxx"}
```

*注意：预期结果中的IP地址和时间戳会根据您的实际网络环境和执行时间而变化。如果代理未配置，第三个示例可能会显示请求失败。*

---
### **2. 结构化日志工具 (`logging_utils.py`)**

`logging_utils.py` 提供了一个强大且灵活的日志记录工具 `PrintMethodClass`，专为自动化脚本设计。它不仅能将日志同时输出到控制台和文件中，还支持动态添加上下文信息，使日志追踪和调试变得异常简单和清晰。

## 核心功能 ✨

- **双重输出**: 日志同时打印到控制台和独立的日志文件 (`./log/脚本名/时间戳.log`)，便于实时查看和事后追溯。
- **动态上下文格式**: 可以随时向日志前缀中添加或移除上下文信息（如用户名、当前任务、代理状态等），让日志内容更丰富、更具可读性。
- **环境配置等级**: 通过环境变量 `log_level` 控制日志输出的详细程度 (DEBUG, INFO, WARNING, ERROR)。
- **自动错误退出**: 在记录 ERROR 级别日志后，可选择自动终止脚本，防止程序在错误状态下继续运行。
- **上下文自动重置**: 可配置在每次打印日志后自动清除临时添加的上下文信息，保持日志清爽，也可设置为手动管理模式。

## 配置

在您的 `/env/config.sh` 文件中，添加以下环境变量来控制全局日志级别。

```shell
# --- 日志等级配置 ---
# 10=DEBUG, 20=INFO, 30=WARNING, 40=ERROR
export log_level="20" # 推荐使用 20 (INFO)
```

## 类与方法详解

### `PrintMethodClass(script_name: str = None, auto_reset: bool = True)`

这是日志工具的核心类。

- **初始化参数**:
  - `script_name` (`str`, optional): 设置一个默认的脚本名称，会显示在每条日志的最前面。
  - `auto_reset` (`bool`, optional): 控制上下文是否自动重置，默认为 `True`。
    - `True`: 调用 `.set()` 添加的上下文信息，在下一次打印日志后会自动清除。
    - `False`: 上下文信息会一直保留，直到你手动调用 `.remove()` 或 `.reset()`。

---
### 上下文管理方法

这些方法用于动态地控制日志前缀中显示的上下文信息。

- **`set(self, key, value)`**
  - **功能**: 添加或更新一个上下文字段。
  - **示例**: `log.set("UserID", "user-001")`

- **`remove(self, key)`**
  - **功能**: 移除一个指定的上下文字段。
  - **示例**: `log.remove("UserID")`

- **`reset(self)`**
  - **功能**: 移除所有通过 `.set()` 添加的临时上下文信息，恢复到初始化状态（`script_name` 和 `Network` 会被重置为 `None`）。

- **`set_script(self, script_name)`**
  - **功能**: 在任何时候修改脚本名称的上下文。

---
### 日志记录方法

- **`debug(self, content)`**: 记录 `DEBUG` 级别的日志。
- **`info(self, content)`**: 记录 `INFO` 级别的日志。
- **`warning(self, content)`**: 记录 `WARNING` 级别的日志。
- **`error(self, content, exit=True)`**: 记录 `ERROR` 级别的日志。
  - `exit` (`bool`): `True` (默认) 表示记录此条日志后，程序将等待10秒然后退出。`False` 表示仅记录日志，程序继续运行。
- **`critical(self, content)`**: 记录 `CRITICAL` 级别的日志，通常也意味着程序需要退出。

---

## 实战测试 Demo

下面的Demo将完整地展示 `PrintMethodClass` 的所有核心功能。

```python
# logging_demo.py
import asyncio
from utils.logging_utils import PrintMethodClass

async def demo_auto_reset():
    """演示 auto_reset=True (默认行为)"""
    log = PrintMethodClass("AutoResetDemo")
    log.info("--- 演示上下文自动重置功能 ---")
    
    # 设置一个临时上下文
    log.set("TaskID", "task-123")
    log.info("这条日志会带上 TaskID。")
    
    # 因为 auto_reset=True，上一次的 TaskID 上下文在这里已经被自动清除了
    log.info("这条日志不会有 TaskID。")

async def demo_manual_reset():
    """演示 auto_reset=False (手动管理)"""
    # 初始化时关闭自动重置
    log = PrintMethodClass("ManualResetDemo", auto_reset=False)
    log.info("--- 演示上下文手动管理功能 ---")

    # 设置一个会话级别的上下文
    log.set("Session", "session-xyz")
    log.info("第1条日志，Session存在。")
    
    # Session 上下文会一直保留
    log.info("第2条日志，Session依然存在。")
    
    # 再添加一个更细粒度的上下文
    log.set("Detail", "processing")
    log.info("第3条日志，同时有 Session 和 Detail。")
    
    # 手动移除一个
    log.remove("Detail")
    log.info("第4条日志，Detail已移除，Session还在。")
    
    # 手动重置所有临时上下文
    log.reset()
    log.info("第5条日志，所有临时上下文都已被重置。")


async def main():
    log = PrintMethodClass("Main")
    log.info("主程序开始。")

    await demo_auto_reset()
    
    log.info("\n" + "="*40 + "\n") # 加个分隔符

    await demo_manual_reset()
    
    log.info("\n--- 演示错误处理 ---")
    try:
        1 / 0
    except Exception as e:
        # exit=False，程序不会退出
        log.error(f"捕获到致命错误: {e}，但程序将继续运行。", exit=False)
        
    log.info("主程序结束。")


if __name__ == "__main__":
    # 假设你的 config.sh 中设置了 export log_level="20"
    asyncio.run(main())
```

### 预期的打印结果

```
[TIME] | INFO     | [Main] : 主程序开始。
[TIME] | INFO     | [AutoResetDemo] : --- 演示上下文自动重置功能 ---
[TIME] | INFO     | [AutoResetDemo] | [TaskID: task-123] : 这条日志会带上 TaskID。
[TIME] | INFO     | [AutoResetDemo] : 这条日志不会有 TaskID。
[TIME] | INFO     | [Main] : 
========================================

[TIME] | INFO     | [ManualResetDemo] : --- 演示上下文手动管理功能 ---
[TIME] | INFO     | [ManualResetDemo] | [Session: session-xyz] : 第1条日志，Session存在。
[TIME] | INFO     | [ManualResetDemo] | [Session: session-xyz] : 第2条日志，Session依然存在。
[TIME] | INFO     | [ManualResetDemo] | [Session: session-xyz] | [Detail: processing] : 第3条日志，同时有 Session 和 Detail。
[TIME] | INFO     | [ManualResetDemo] | [Session: session-xyz] : 第4条日志，Detail已移除，Session还在。
[TIME] | INFO     | [ManualResetDemo] : 第5条日志，所有临时上下文都已被重置。
[TIME] | INFO     | [Main] : 
--- 演示错误处理 ---
[TIME] | ERROR    | [Main] : 捕获到致命错误: division by zero，但程序将继续运行。
[TIME] | INFO     | [Main] : 主程序结束。
```

---
### **3. 异步并发控制器 (`concurrency_utils.py`)**

`concurrency_utils.py` 模块提供了一套强大的工具，用于优雅地管理和执行大量异步任务。它的核心是 `RunMethod` 类，能够以指定的并发数（即“线程数”）分批执行任务，并能控制每批任务之间的等待间隔，是防止API速率限制、管理系统资源和构建高效率自动化脚本的关键。

## 核心功能 ✨

- **并发控制**: 通过 `Semaphore` 机制，精确控制同时运行的异步任务数量，避免瞬间请求过多导致目标服务器过载或IP被封锁。
- **任务批处理**: 自动将大量任务分割成多个小批次，逐批执行，使代码逻辑更清晰。
- **可配置等待**: 可以在每批任务执行完毕后，设置一个固定的等待时间，用于模拟人类操作间隔或应对更严格的速率限制。
- **标准化出入参**: 通过 `ReqConcParam` 和 `ReqConcResult` 数据类，将任务的配置和结果进行结构化封装，使代码更易于维护和理解。
- **灵活的参数传递**: 支持向并发执行的函数传递固定的额外参数。

## 核心类与方法详解

### 数据类 (Data Classes)

#### `ReqConcParam(func, task, thread, wait, **kwargs)`
这是一个配置类，用于将并发任务所需的所有参数打包成一个对象。

- **参数**:
  - `func` (`Callable`): **必需**。您想要并发执行的那个异步函数。
  - `task` (`List` or `Tuple`): **必需**。一个包含所有任务参数的可迭代对象。运行器会遍历这个列表，并将其中每一个元素作为 `func` 的第一个参数传入。
  - `thread` (`int`): **必需**。最大并发数，即同一时间最多有多少个 `func` 在运行。
  - `wait` (`int` or `float`, optional): 每执行完一批任务后的等待秒数，默认为 `0` (不等待)。
  - `**kwargs`: (optional) 任意数量的关键字参数，这些参数会作为固定参数传递给每一次 `func` 的调用。

#### `ReqConcResult(result, task)`
这是 `ReqConcRun` 运行器返回的每一项结果的包装类，便于您将结果与原始任务对应起来。

- **属性**:
  - `result` (`any`): `func` 函数执行后的返回值。
  - `task` (`any`): 触发这次执行的原始任务项（来自 `ReqConcParam` 中的 `task` 列表）。

### 主运行器类 `RunMethod`

这个类包含了执行并发任务的核心方法。所有方法都是类方法，通过 `RunMethod.method_name()` 直接调用。

#### `async def ReqConcRun(cls, param: ReqConcParam)`
这是最常用、最推荐的并发运行器。它是一个**异步生成器**，你需要使用 `async for` 来迭代它返回的批处理结果。

- **功能**: 接收一个 `ReqConcParam` 对象，根据其配置（并发数、等待时间等）来执行任务列表。
- **返回**: 它会逐批 `yield` 结果。每一批的结果是一个列表，列表中的每个元素都是一个 `ReqConcResult` 对象。

---

## 实战测试 Demo

下面的Demo将完整地展示如何使用 `ReqConcRun` 来并发执行一组模拟的API请求，并处理返回结果。

```python
# concurrency_demo.py
import asyncio
import time
from typing import Dict, List, Coroutine

from utils.concurrency_utils import RunMethod, ReqConcParam, ReqConcResult
from utils.logging_utils import PrintMethodClass

# 初始化日志记录器，以便清晰地看到执行流程
log = PrintMethodClass("ConcurrencyDemo")

# 1. 定义一个模拟耗时的 "工人" 异步函数
# 它接收一个任务项（字典）和一些额外的固定参数
async def worker_task(item: Dict, session_id: str, timeout: int) -> Dict:
    """
    模拟一个需要执行的网络请求或IO密集型任务。
    """
    task_id = item['id']
    delay = item['delay']
    log.info(f"[会话: {session_id}] 任务 {task_id} 开始，预计耗时 {delay} 秒...")
    
    # 模拟实际工作
    await asyncio.sleep(delay)
    
    log.info(f"任务 {task_id} 完成！")
    return {"task_id": task_id, "status": "ok", "completed_at": time.time()}

async def main():
    log.info("--- 准备并发任务 ---")
    
    # 2. 准备一批需要处理的任务数据
    tasks_to_run: List[Dict] = [
        {'id': 1, 'delay': 2.0}, {'id': 2, 'delay': 1.0}, {'id': 3, 'delay': 2.5},
        {'id': 4, 'delay': 1.5}, {'id': 5, 'delay': 0.5}, {'id': 6, 'delay': 3.0},
        {'id': 7, 'delay': 1.0},
    ]

    # 3. 使用 ReqConcParam 对任务进行配置
    conc_params = ReqConcParam(
        func=worker_task,       # 要执行的函数
        task=tasks_to_run,      # 任务列表
        thread=3,               # 最大并发数为 3
        wait=2,                 # 每批任务完成后，等待 2 秒
        session_id="session-abc", # 这是一个将传递给每个worker_task的固定参数
        timeout=30              # 同上
    )
    log.info(f"任务配置完毕: {len(tasks_to_run)}个任务，并发数{conc_params.thread}，批间等待{conc_params.wait}秒。")

    # 4. 使用 async for 迭代运行器，并处理每批返回的结果
    start_time = time.time()
    async for batch_result in RunMethod.ReqConcRun(conc_params):
        log.info(f"--- 一批任务执行完毕 (耗时: {time.time() - start_time:.2f}s) ---")
        for res in batch_result:
            # res 是 ReqConcResult 对象
            log.info(f"  [结果] 任务 {res.task['id']} 已完成, 返回状态: {res.result['status']}")
    
    end_time = time.time()
    log.info(f"\n所有任务执行完毕，总耗时: {end_time - start_time:.2f}s")


if __name__ == "__main__":
    asyncio.run(main())
```

### 预期的打印结果

```
[TIME] | INFO     | [ConcurrencyDemo] : --- 准备并发任务 ---
[TIME] | INFO     | [ConcurrencyDemo] : 任务配置完毕: 7个任务，并发数3，批间等待2秒。
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 1 开始，预计耗时 2.0 秒...
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 2 开始，预计耗时 1.0 秒...
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 3 开始，预计耗时 2.5 秒...
[TIME] | INFO     | [ConcurrencyDemo] : 任务 2 完成！
[TIME] | INFO     | [ConcurrencyDemo] : 任务 1 完成！
[TIME] | INFO     | [ConcurrencyDemo] : 任务 3 完成！
[TIME] | INFO     | [ConcurrencyDemo] : --- 一批任务执行完毕 (耗时: 2.52s) ---
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 1 已完成, 返回状态: ok
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 2 已完成, 返回状态: ok
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 3 已完成, 返回状态: ok
(此处会等待2秒)
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 4 开始，预计耗时 1.5 秒...
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 5 开始，预计耗时 0.5 秒...
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 6 开始，预计耗时 3.0 秒...
[TIME] | INFO     | [ConcurrencyDemo] : 任务 5 完成！
[TIME] | INFO     | [ConcurrencyDemo] : 任务 4 完成！
[TIME] | INFO     | [ConcurrencyDemo] : 任务 6 完成！
[TIME] | INFO     | [ConcurrencyDemo] : --- 一批任务执行完毕 (耗时: 7.55s) ---
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 4 已完成, 返回状态: ok
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 5 已完成, 返回状态: ok
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 6 已完成, 返回状态: ok
(此处会等待2秒)
[TIME] | INFO     | [ConcurrencyDemo] : [会话: session-abc] 任务 7 开始，预计耗时 1.0 秒...
[TIME] | INFO     | [ConcurrencyDemo] : 任务 7 完成！
[TIME] | INFO     | [ConcurrencyDemo] : --- 一批任务执行完毕 (耗时: 10.57s) ---
[TIME] | INFO     | [ConcurrencyDemo] :   [结果] 任务 7 已完成, 返回状态: ok
[TIME] | INFO     | [ConcurrencyDemo] : 
所有任务执行完毕，总耗时: 10.57s
```

*注意：预期结果中的总耗时和日志时间戳会因您的机器性能而略有不同，但执行的批次和顺序逻辑将保持一致。*

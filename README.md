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

---
### **4. 环境配置工具 (`env_utils.py`)**

`env_utils.py` 提供了一个强大的静态类 `EnvMethod`，用于以更智能、更结构化的方式读取和管理项目配置。它远比 `os.getenv` 更强大，能够自动处理类型转换、提供默认值，并支持将环境变量批量加载到类的属性中，极大地提升了代码的可读性和健壮性。

## 核心功能 ✨

- **智能类型转换**: 自动将环境变量中的 `"true"`/`"false"` 转换为布尔值，将 `"[...]"`, `"{...}"` 格式的字符串解析为Python列表或字典。
- **多种列表格式支持**: 支持以 `|` 分隔的字符串列表（如 `"a|b|c"`）和JSON格式的列表。
- **类型强制转换**: 提供 `codeInt` 等参数，用于将字符串强制转换为整数或整数列表。
- **安全的默认值**: 当环境变量不存在时，可以安全地返回指定的默认值。
- **结构化配置加载**: 提供了两种强大的方法 (`load_config_from_env` 和 `checkEnv`)，可以将分散的环境变量自动、安全地加载到一个配置类的实例中，让配置管理井井有条。
- **反向写入支持**: 提供了 `newEnv` 和 `upEnv` 方法，用于通过代码修改 `config.sh` 文件（不常用）。

---
## 方法详解

`EnvMethod` 中的所有方法都是静态方法，通过 `EnvMethod.method_name()`直接调用。

### `readEnv(key, default=None, codeInt=False, codeList=False)`

这是最基础也是最核心的读取方法。

- **功能**: 智能地读取单个环境变量并进行类型转换。
- **参数**:
  - `key` (`str`): **必需**。环境变量的名称。
  - `default` (`any`, optional): 当环境变量不存在或值为空时，返回此默认值。
  - `codeInt` (`bool`, optional): 设置为 `True` 时，会尝试将读取到的值（或列表中的每一项）转换为整数。如果转换失败，会返回 `default` 值。
  - `codeList` (`bool`, optional): 设置为 `True` 时，即使字符串中不包含 `|`，也会强制将其视为只有一个元素的列表。

---
### `load_config_from_env(obj_self, config_obj, err_exit=True)`

**（推荐的配置加载方式）** 这是一种非常优雅的配置加载模式，利用Python的类型提示和类变量。

- **功能**: 自动读取并加载一系列环境变量到目标对象的属性上。
- **工作流程**:
    1.  它会查看 `config_obj` 这个配置模板类。
    2.  通过 `config_obj.env_map` 字典找到 **类属性名** 和 **环境变量名** 的映射关系。
    3.  通过 `config_obj.__annotations__` (类型提示) 来决定如何转换变量类型（例如 `int`, `List[int]`）。
    4.  读取环境变量，转换类型，然后用 `setattr` 将值赋给 `obj_self` (通常是 `self`)。
- **参数**:
  - `obj_self` (`object`): 要被设置属性的目标对象。
  - `config_obj` (`object`): 一个专门用于定义配置结构的类的实例。这个类需要包含：
    - `env_map` (dict): 映射表。
    - `__annotations__` (类型提示): 用于自动类型转换。
    - 属性的默认值。

---
### `checkEnv(obj_self, config, err_exit=True)`

这是一种更为传统的配置加载方式，通过一个列表来定义所有需要加载的变量。

- **功能**: 根据一个配置列表，读取环境变量并设置到目标对象的属性上。
- **参数**:
  - `obj_self` (`object`): 要被设置属性的目标对象。
  - `config` (`List[Dict]`): 一个包含多个字典的列表，每个字典定义一个要加载的变量，格式如下：
    ```python
    {
        "env_key": "YOUR_ENV_VAR_NAME",  # 环境变量名
        "attribute": "your_attribute_name", # 要设置到的对象属性名
        "type": int,  # 目标类型，如 str, int, List[int]
        "default": 0  # 默认值
    }
    ```

---

## 实战测试 Demo

下面的Demo将完整地展示 `readEnv` 的智能转换能力，以及两种配置加载方法 `load_config_from_env` 和 `checkEnv` 的具体用法。

```python
# env_demo.py
import os
from typing import List, get_origin, get_args
from utils.env_utils import EnvMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("EnvDemo")

# 1. 模拟在 config.sh 中设置环境变量
os.environ['DEMO_STR'] = "hello world"
os.environ['DEMO_INT'] = "123"
os.environ['DEMO_LIST_PIPE'] = "a|b|c"
os.environ['DEMO_JSON_LIST'] = '[1, 2, 3]'
os.environ['DEMO_BOOL'] = 'true'
os.environ['DEMO_NOT_EXIST'] = ''

# --- Demo Part 1: 测试 readEnv 的各种情况 ---
def demo_read_env():
    log.info("--- 测试 readEnv ---")
    str_val = EnvMethod.readEnv("DEMO_STR")
    int_val = EnvMethod.readEnv("DEMO_INT", 0, codeInt=True)
    list_pipe_val = EnvMethod.readEnv("DEMO_LIST_PIPE")
    json_list_val = EnvMethod.readEnv("DEMO_JSON_LIST")
    bool_val = EnvMethod.readEnv("DEMO_BOOL")
    default_val = EnvMethod.readEnv("DEMO_NOT_EXIST", "我是默认值")

    log.info(f"读取字符串: '{str_val}' (类型: {type(str_val)})")
    log.info(f"读取整数: {int_val} (类型: {type(int_val)})")
    log.info(f"读取管道列表: {list_pipe_val} (类型: {type(list_pipe_val)})")
    log.info(f"读取JSON列表: {json_list_val} (类型: {type(json_list_val)})")
    log.info(f"读取布尔值: {bool_val} (类型: {type(bool_val)})")
    log.info(f"读取默认值: '{default_val}' (类型: {type(default_val)})")

# --- Demo Part 2: 测试 load_config_from_env (推荐方式) ---
def demo_load_from_class():
    log.info("\n--- Demo 2: 测试 load_config_from_env (推荐方式) ---")
    
    # a. 定义一个配置模板类
    class AppConfig:
        # 映射表：将类属性名映射到环境变量名
        env_map = {
            'str_setting': 'DEMO_STR',
            'int_setting': 'DEMO_INT',
            'list_setting': 'DEMO_LIST_PIPE'
        }
        # b. 定义属性的类型提示和默认值
        str_setting: str = "default_string"
        int_setting: int = 0
        list_setting: List[str] = []

    # c. 在你的主程序类中使用
    class MyApp:
        def __init__(self):
            # 将配置加载到 self 的属性上
            EnvMethod.load_config_from_env(self, AppConfig())
    
    app = MyApp()
    log.info(f"加载后 app.str_setting = '{app.str_setting}'")
    log.info(f"加载后 app.int_setting = {app.int_setting}")
    log.info(f"加载后 app.list_setting = {app.list_setting}")

# --- Demo Part 3: 测试 checkEnv (备选方式) ---
def demo_check_env():
    log.info("\n--- Demo 3: 测试 checkEnv (备选方式) ---")
    
    # a. 定义一个配置清单
    CONFIG_LIST = [
        {"env_key": "DEMO_STR", "attribute": "str_attr", "type": str, "default": "d"},
        {"env_key": "DEMO_INT", "attribute": "int_attr", "type": int, "default": 0},
    ]

    # b. 在你的主程序类中使用
    class AnotherApp:
        def __init__(self):
            self.str_attr = ""
            self.int_attr = 0
            EnvMethod.checkEnv(self, CONFIG_LIST)
            
    app = AnotherApp()
    log.info(f"加载后 app.str_attr = '{app.str_attr}'")
    log.info(f"加载后 app.int_attr = {app.int_attr}")

# --- 运行所有Demo ---
demo_read_env()
demo_load_from_class()
demo_check_env()
```

### 预期的打印结果

```
[TIME] | INFO     | [EnvDemo] : --- 测试 readEnv ---
[TIME] | INFO     | [EnvDemo] : 读取字符串: 'hello world' (类型: <class 'str'>)
[TIME] | INFO     | [EnvDemo] : 读取整数: 123 (类型: <class 'int'>)
[TIME] | INFO     | [EnvDemo] : 读取管道列表: ['a', 'b', 'c'] (类型: <class 'list'>)
[TIME] | INFO     | [EnvDemo] : 读取JSON列表: [1, 2, 3] (类型: <class 'list'>)
[TIME] | INFO     | [EnvDemo] : 读取布尔值: True (类型: <class 'bool'>)
[TIME] | INFO     | [EnvDemo] : 读取默认值: '我是默认值' (类型: <class 'str'>)
[TIME] | INFO     | [EnvDemo] : 
--- Demo 2: 测试 load_config_from_env (推荐方式) ---
[TIME] | INFO     | [EnvDemo] : 加载后 app.str_setting = 'hello world'
[TIME] | INFO     | [EnvDemo] : 加载后 app.int_setting = 123
[TIME] | INFO     | [EnvDemo] : 加载后 app.list_setting = ['a', 'b', 'c']
[TIME] | INFO     | [EnvDemo] : 
--- Demo 3: 测试 checkEnv (备选方式) ---
[TIME] | INFO     | [EnvDemo] : 加载后 app.str_attr = 'hello world'
[TIME] | INFO     | [EnvDemo] : 加载后 app.int_attr = 123
```

---
### **5. 异步进程管理器 (`script_executor.py`)**

`script_executor.py` 提供了一个强大的 `ProcessManager` 类，用于以**异步、非阻塞**的方式启动、监控和管理外部脚本或系统命令。这对于需要执行耗时任务（如运行一个复杂的Python脚本、Node.js应用或Shell脚本）而又不想阻塞主程序（例如一个正在监听网络请求的服务器）的场景至关重要。

## 核心功能 ✨

- **非阻塞执行**: 启动的脚本或命令会作为一个独立的子进程在后台运行，不会阻塞主程序的 `asyncio` 事件循环。
- **智能脚本识别**: `run_script` 方法能根据文件后缀（`.py`, `.js`, `.sh`）自动选择合适的解释器来执行。
- **完整的生命周期管理**: 可以启动、监控、等待、甚至提前终止子进程。
- **丰富的过程信息**: `ProcessInfo` 数据类完整地记录了进程的PID、状态、开始/结束时间、返回码以及完整的 `stdout` (标准输出) 和 `stderr` (标准错误) 。
- **回调支持**: 可以在进程执行完毕后自动触发一个回调函数，用于处理结果或启动下一个任务，实现任务链。
- **健壮的错误处理**: 能捕获子进程的错误输出和非零返回码，便于主程序进行判断和处理。

## 核心类与方法详解

### `ProcessInfo` 数据类
这是 `ProcessManager` 中每个进程的信息载体，当您等待一个进程结束后，获取到的就是这个对象。

- **核心属性**:
  - `pid` (`int`): 进程的唯一标识符。
  - `script_path` (`str`): 被执行的脚本路径。
  - `status` (`str`): 进程的当前状态 (`running`, `completed`, `failed`, `killed`)。
  - `start_time` / `end_time` (`datetime`): 进程的开始与结束时间。
  - `return_code` (`int`): 进程的退出码。通常 `0` 表示成功，非 `0` 表示有错误。
  - `stdout` (`str`): 子进程的标准输出内容。
  - `stderr` (`str`): 子进程的标准错误内容。

### `ProcessManager` 类
这是与进程交互的主控制器。

#### `async def run_script(self, script_path, run_method='auto', args=[], callback=None, **kwargs)`
最常用的方法，用于异步执行一个脚本文件。

- **参数**:
  - `script_path` (`str`): **必需**。脚本的路径。
  - `run_method` (`str`, optional): 运行方式，`'auto'` (默认) 会根据文件后缀自动检测。也可以强制指定为 `'python'`, `'node'`, `'bash'` 等。
  - `args` (`List[str]`, optional): 一个字符串列表，作为命令行参数传递给脚本。
  - `callback` (`Callable`, optional): 一个函数，当进程执行结束后会被自动调用。该函数应接收一个 `ProcessInfo` 对象作为其唯一参数。

- **返回**:
  - `int`: 新创建的子进程的PID。

#### `async def run_command(self, command: List[str], ...)`
更底层的执行方法，用于运行任何系统命令。

- **参数**:
  - `command` (`List[str]`): **必需**。一个包含命令及其所有参数的列表，例如 `['ls', '-l', '/tmp']`。

- **返回**:
  - `int`: 新创建的子进程的PID。

#### `async def wait_for_process(self, pid: int, timeout: float = None) -> ProcessInfo`
异步地等待一个已启动的进程执行完毕。

- **功能**: 调用此方法会使当前协程暂停，直到指定的 `pid` 进程结束，然后才会继续执行。
- **参数**:
  - `pid` (`int`): `run_script` 或 `run_command` 返回的进程ID。
  - `timeout` (`float`, optional): 最大等待时间（秒）。如果超时，会引发 `TimeoutError`。
- **返回**:
  - `ProcessInfo`: 包含该进程完整执行结果的信息对象。

#### `async def kill_process(self, pid: int) -> bool`
优雅地终止一个正在运行的进程。

---

## 实战测试 Demo

下面的Demo将完整地展示如何使用 `ProcessManager` 启动一个带参数的Python脚本，在不阻塞主程序的情况下等待它完成，并通过回调函数和 `wait_for_process` 两种方式获取其最终结果。

```python
# executor_demo.py
import asyncio
import sys
from utils.script_executor import ProcessManager, ProcessInfo
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ExecutorDemo")

# 1. 为了演示，我们先动态创建一个简单的Python脚本文件
DUMMY_SCRIPT_PATH = "my_test_script.py"
with open(DUMMY_SCRIPT_PATH, "w", encoding="utf-8") as f:
    f.write('import time, sys\n')
    f.write('print(f"脚本开始运行，接收到参数: {sys.argv[1:]}")\n')
    f.write('sys.stderr.write("这是一条错误输出信息\\n")\n') # 模拟错误输出
    f.write('time.sleep(3)\n')
    f.write('print("脚本运行结束。")\n')
    f.write('sys.exit(0)\n') # 模拟正常退出

# 2. 定义一个回调函数，它将在脚本结束后被自动调用
def my_process_callback(info: ProcessInfo):
    """这个函数会在子进程结束后被触发"""
    log.info(f"--- 监听到进程 {info.pid} 结束（回调函数触发） ---")
    log.info(f"  - 脚本路径: {info.script_path}")
    log.info(f"  - 最终状态: {info.status}")
    log.info(f"  - 退出码: {info.return_code}")
    log.info("--- 回调结束 ---")

async def main():
    # 3. 实例化进程管理器
    manager = ProcessManager()
    
    log.info("准备以非阻塞方式运行外部Python脚本...")
    
    # 4. 使用 run_script 启动脚本，并传入参数和回调函数
    pid = await manager.run_script(
        script_path=DUMMY_SCRIPT_PATH,
        args=['param1', 'value1'], # 传递给脚本的命令行参数
        callback=my_process_callback
    )
    log.info(f"脚本已在后台启动，其进程PID为: {pid}")
    
    # 5. 在等待子进程的同时，主程序可以继续执行其他任务
    log.info("主程序没有被阻塞，现在执行其他异步任务...")
    await asyncio.sleep(1) # 模拟其他工作
    log.info("主程序的其他任务已完成，现在开始等待子进程结束。")

    # 6. 使用 wait_for_process 等待脚本执行完成，并获取最终的详细信息
    try:
        final_info = await manager.wait_for_process(pid)
        
        log.info(f"\n--- 主程序通过 wait_for_process 获取到最终结果 (PID: {final_info.pid}) ---")
        log.info(f"状态: {final_info.status}")
        log.info(f"返回码: {final_info.return_code}")
        log.info(f"开始时间: {final_info.start_time.strftime('%Y-%m-%d %H:%M:%S')}")
        log.info(f"结束时间: {final_info.end_time.strftime('%Y-%m-%d %H:%M:%S')}")
        log.info(f"标准输出 (stdout):\n---\n{final_info.stdout.strip()}\n---")
        log.info(f"标准错误 (stderr):\n---\n{final_info.stderr.strip()}\n---")

    except Exception as e:
        log.error(f"等待进程时发生错误: {e}")

if __name__ == "__main__":
    asyncio.run(main())
```

### 预期的打印结果

```
[TIME] | INFO     | [ExecutorDemo] : 准备以非阻塞方式运行外部Python脚本...
[TIME] | INFO     | [ExecutorDemo] : 执行命令: <...>/python my_test_script.py param1 value1
[TIME] | INFO     | [ExecutorDemo] : 命令已启动, PID: <pid_number>
[TIME] | INFO     | [ExecutorDemo] | [PID: <pid_number>] : 监控任务已启动。
[TIME] | INFO     | [ExecutorDemo] : 脚本已在后台启动，其进程PID为: <pid_number>
[TIME] | INFO     | [ExecutorDemo] : 主程序没有被阻塞，现在执行其他异步任务...
[TIME] | INFO     | [ExecutorDemo] : 主程序的其他任务已完成，现在开始等待子进程结束。
[TIME] | INFO     | [ExecutorDemo] : 正在等待进程 [<pid_number>] 完成...
[TIME] | INFO     | [ExecutorDemo] | [PID: <pid_number>] : 命令执行完成。
[TIME] | INFO     | [ExecutorDemo] : --- 监听到进程 <pid_number> 结束（回调函数触发） ---
[TIME] | INFO     | [ExecutorDemo] :   - 脚本路径: my_test_script.py
[TIME] | INFO     | [ExecutorDemo] :   - 最终状态: completed
[TIME] | INFO     | [ExecutorDemo] :   - 退出码: 0
[TIME] | INFO     | [ExecutorDemo] : --- 回调结束 ---
[TIME] | INFO     | [ExecutorDemo] | [PID: <pid_number>] : 监控任务已结束。
[TIME] | INFO     | [ExecutorDemo] : 进程 [<pid_number>] 已完成，等待结束。
[TIME] | INFO     | [ExecutorDemo] : 
--- 主程序通过 wait_for_process 获取到最终结果 (PID: <pid_number>) ---
[TIME] | INFO     | [ExecutorDemo] : 状态: completed
[TIME] | INFO     | [ExecutorDemo] : 返回码: 0
[TIME] | INFO     | [ExecutorDemo] : 开始时间: 2025-07-10 13:20:35
[TIME] | INFO     | [ExecutorDemo] : 结束时间: 2025-07-10 13:20:38
[TIME] | INFO     | [ExecutorDemo] : 标准输出 (stdout):
---
脚本开始运行，接收到参数: ['param1', 'value1']
脚本运行结束。
---
[TIME] | INFO     | [ExecutorDemo] : 标准错误 (stderr):
---
这是一条错误输出信息
---
```
*注意：结果中的PID和时间戳会根据您的实际执行情况而变化。*

---
 ### **6. 高精度异步任务调度器 (`time_scheduler.py`)**

`time_scheduler.py` 提供了一个专为时间敏感型任务设计的 `ServerTimeScheduler` 类。它的核心价值在于，它不依赖可能存在偏差的本地计算机时钟，而是通过一个您指定的函数（如获取京东服务器时间的API）来**与服务器时间进行同步**。这使得它非常适合用于执行需要精确到秒甚至毫秒的定时任务，例如电商平台的“秒杀”活动。

## 核心功能 ✨

- **服务器时间同步**: 彻底解决本地时钟不准导致的任务触发延迟或提前问题。
- **高精度等待**: 能够计算并等待到指定时、分、秒的精确时刻，精度取决于网络延迟。
- **灵活的调度模式**:
    - `wait_until`: 用于“抢购”类场景，等待一个精确的时间点。
    - `sleep_until_next_active_period`: 用于“周期任务”类场景，等待进入下一个指定的活跃时间段。
- **异步非阻塞**: 完全基于 `asyncio`，在等待期间不会阻塞事件循环，可以同时执行其他任务。

## 核心类与方法详解

### `ServerTimeScheduler(func: Union[Callable, Awaitable])`

这是您将要使用的核心调度器类。

- **初始化参数**:
  - `func` (`Callable` or `Awaitable`): **必需**。这是一个用于获取**服务器时间**的异步函数。此函数必须返回一个 **13位的毫秒级时间戳** (整数或浮点数)。在本项目中，通常传入 `JdApiClient().Jd_Time`。调度器会通过调用此函数来校准自己的时钟。

---
### 主要调度方法

#### `async def wait_until(self, hour: int, minute: int = 0, second: int = 0, microsecond: int = 0) -> float`

这是最高精度的方法，用于等待一个**精确的未来时间点**。

- **功能**: 计算当前时间到下一个指定 `时:分:秒` 的时间差，并异步休眠直到那一刻。如果指定的时间在今天已经过去，它会自动计算并等待到**第二天的这个时间**。
- **参数**:
  - `hour` (`int`): 目标小时 (0-23)。
  - `minute` (`int`, optional): 目标分钟 (0-59)。
  - `second` (`int`, optional): 目标秒 (0-59)。
  - `microsecond` (`int`, optional): 目标微秒。
- **返回**:
  - `float`: 它实际等待到的目标时刻的服务器时间戳（秒）。

#### `async def sleep_until_next_active_period(self, active_hours: Optional[List[str]] = None) -> None`

用于等待进入下一个**活跃的时间段**，适合不需要精确到秒的周期性任务。

- **功能**: 检查当前时间是否在 `active_hours` 定义的时间段内。如果不在，则计算并休眠，直到下一个最近的活跃时间段开始。
- **参数**:
  - `active_hours` (`List[str]`, optional): 一个定义了活跃小时范围的字符串列表。格式说明：
    - `"8"`: 表示 8:00:00 到 8:59:59 这个小时都算活跃。
    - `"10-14"`: 表示从 10:00:00 到 14:59:59 都算活跃。
    - 如果不提供此参数，则默认等待到下一个整点小时。

---

## 实战测试 Demo

下面的Demo将完整地展示如何初始化调度器，并使用 `wait_until` 方法实现一个精确的定时任务。

```python
# scheduler_demo.py
import asyncio
from datetime import datetime, timedelta
from utils.time_scheduler import ServerTimeScheduler
from utils.jd_api import JdApiClient # 导入京东API客户端以获取服务器时间
from utils.logging_utils import PrintMethodClass

# 初始化日志记录器
log = PrintMethodClass("SchedulerDemo")

# 1. 准备一个高精度任务
async def high_precision_task():
    """这个任务将在精确的时刻被触发"""
    log.info(f"任务被触发！精确的服务器时间大约是: {datetime.now().strftime('%Y-%m-%d %H:%M:%S.%f')}")

# 主执行函数
async def main():
    log.info("--- 初始化高精度调度器 ---")
    # 2. 实例化 JdApiClient 以便调用其时间同步方法
    jd_client = JdApiClient()
    
    # 3. 实例化调度器，并将 jd_client.Jd_Time 作为时间源函数传入
    scheduler = ServerTimeScheduler(func=jd_client.Jd_Time)
    
    # 4. 手动同步一次时间并打印，以观察其工作原理
    # 这一步在实际使用 wait_until 时会自动调用，这里仅为演示
    try:
        server_time_ms = await scheduler.sync_time()
        server_dt = datetime.fromtimestamp(server_time_ms / 1000)
        log.info(f"手动同步时间成功，当前服务器时间约为: {server_dt.strftime('%Y-%m-%d %H:%M:%S')}")
    except Exception as e:
        log.error(f"时间同步失败，请检查网络或时间源函数！错误: {e}")
        return

    # 5. 设置一个将在5秒后执行的定时任务
    now = datetime.now()
    target_time = now + timedelta(seconds=5)
    log.info(f"当前本地时间: {now.strftime('%H:%M:%S')}, 准备等待到未来的: {target_time.strftime('%H:%M:%S')}")
    
    # 6. 调用 wait_until，它会精确计算需要休眠的时间
    await scheduler.wait_until(
        hour=target_time.hour,
        minute=target_time.minute,
        second=target_time.second
    )
    
    # 7. 当 wait_until 执行完毕后，立刻执行我们的高精度任务
    await high_precision_task()
    
    log.info("Demo 结束。")


if __name__ == "__main__":
    asyncio.run(main())
```

### 预期的打印结果

```
[TIME] | INFO     | [SchedulerDemo] : --- 初始化高精度调度器 ---
[TIME] | INFO     | [SchedulerDemo] | [jdtime] : 手动同步时间成功，当前服务器时间约为: 2025-07-10 13:24:00
[TIME] | INFO     | [SchedulerDemo] : 当前本地时间: 13:24:00, 准备等待到未来的: 13:24:05
[TIME] | INFO     | [SchedulerDemo] | [jdtime] : 需要等待 4.98 秒，直到 2025-07-10 13:24:05
(程序会在这里安静地等待大约5秒)
[TIME] | INFO     | [SchedulerDemo] : 任务被触发！精确的服务器时间大约是: 2025-07-10 13:24:05.001234
[TIME] | INFO     | [SchedulerDemo] : Demo 结束。
```
*注意：结果中的时间戳会根据您的实际执行时间而变化，但 `wait_until` 会确保 `high_precision_task` 在接近目标时间点 `XX:XX:05` 时被精确触发。*

---
### **7. User-Agent 生成器 (`user_agent_generator.py`)**

`user_agent_generator.py` 模块提供了一个强大的工具，用于生成海量、真实的移动设备User-Agent字符串。在进行网络爬取或自动化操作时，使用随机且真实的UA是避免被目标网站识别和封锁的关键一步。本模块内置了主流手机品牌（苹果、华为、小米、三星等）的上千种设备型号，并特别包含了针对京东App的专用UA生成器。

## 核心功能 ✨

- **海量设备库**: 内置了来自 Apple, Google, Samsung, Xiaomi, HUAWEI, OPPO, OnePlus, VIVO, MEIZU 等主流厂商的设备型号、代号和内部编码，确保生成的设备信息真实可信。
- **结构化设备信息**: 获取设备信息时，返回的是一个结构清晰的 `GetPhoneResponse` 对象，而不仅仅是一个字符串，便于您提取和使用其中的任何部分（如品牌、型号等）。
- **通用UA构建**: 您可以利用获取到的设备信息，轻松构建标准的移动端浏览器User-Agent。
- **京东App专用UA**: `JdUserAgentGenerator` 类可以生成与京东官方App行为一致的、包含复杂加密参数的UA，是进行京东相关自动化任务的利器。

## 核心类与方法详解

### `PhoneModel` 类

这是一个静态工具类，用于从内置的设备库中随机获取设备信息。

#### `get_phone_models(brand: Optional[str] = None) -> GetPhoneResponse`
- **功能**: 随机选择一个设备型号并返回其详细信息。
- **参数**:
  - `brand` (`str`, optional): **可选参数**。用于指定手机品牌，如 `"Apple"`, `"Xiaomi"`, `"HUAWEI"`。如果留空，则从所有品牌中随机抽取。
- **返回**:
  - `GetPhoneResponse`: 一个包含设备详细信息的数据对象。

### `GetPhoneResponse` 数据类

这是 `get_phone_models` 方法返回的对象类型。

- **属性**:
  - `brand` (`str`): 品牌名称，如 `"Xiaomi"`。
  - `name` (`str`): 设备系列名称，如 `"Xiaomi 14 Pro"`。
  - `code_name` (`str`): 设备的开发代号，如 `"shennong"`。
  - `model_number` (`str`): 具体的设备型号，如 `"23116PN5BC"`。
  - `phone_name` (`str`): 设备的市场名称，如 `"Xiaomi 14 Pro / Xiaomi 14 Pro 钛金属版"`。

### `JdUserAgentGenerator` 类

这是一个专门用于生成京东App User-Agent的类。

#### `__init__(self, clientVersion: Optional[str] = None, build: Optional[str] = None)`
- **功能**: 初始化京东UA生成器。
- **参数**:
  - `clientVersion` (`str`, optional): 指定京东App的版本号，如 `"12.3.4"`。如果留空，会使用内置的默认值。
  - `build` (`str`, optional): 指定京东App的构建号，如 `"100574"`。如果留空，会使用内置的默认值。

#### `jd_app_ua(self, name: Literal["jd", "jdh"]) -> JdUserAgentResponse`
- **功能**: 生成京东App的User-Agent对。
- **参数**:
  - `name` (`str`): **必需**。指定要生成的UA类型，`"jd"` 代表京东主站App，`"jdh"` 代表京东健康App。
- **返回**:
  - `JdUserAgentResponse`: 一个包含两种格式UA的数据对象。

### `JdUserAgentResponse` 数据类

这是 `jd_app_ua` 方法返回的对象类型。

- **属性**:
  - `app` (`str`): 用于WebView或一般HTTP请求的完整UA字符串。
  - `okhttp` (`str`): 用于模拟底层 `okhttp` 库请求的专用UA字符串。

---

## 实战测试 Demo

下面的Demo将完整地展示如何生成通用UA和专用的京东App UA。

```python
# ua_generator_demo.py
from utils.user_agent_generator import PhoneModel, JdUserAgentGenerator
from utils.logging_utils import PrintMethodClass
import random

# 初始化日志记录器
log = PrintMethodClass("UADemo")

def main():
    log.info("--- 1. 获取一个完全随机的设备信息 ---")
    random_device = PhoneModel.get_phone_models()
    log.info(f"随机抽取的设备品牌: {random_device.brand}")
    log.info(f"设备市场名称: {random_device.phone_name}")
    log.info(f"具体型号: {random_device.model_number}")
    log.info(f"开发代号: {random_device.code_name}")
    
    log.info("\n--- 2. 根据随机设备信息构建一个通用的浏览器UA ---")
    android_version = random.choice(["12", "13", "14"])
    generic_ua = (
        f"Mozilla/5.0 (Linux; Android {android_version}; {random_device.model_number}) "
        f"AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Mobile Safari/537.36"
    )
    log.info(f"构建的通用UA: {generic_ua}")

    log.info("\n--- 3. 获取一个指定品牌（华为）的设备信息 ---")
    huawei_device = PhoneModel.get_phone_models(brand="HUAWEI")
    log.info(f"随机华为设备: {huawei_device.name} ({huawei_device.model_number})")

    log.info("\n--- 4. 生成京东App专用User-Agent ---")
    # 初始化京东UA生成器，可以指定版本号
    jd_ua_gen = JdUserAgentGenerator(clientVersion="12.3.4", build="100500")
    
    # 生成京东主站App的UA
    jd_ua_pair = jd_ua_gen.jd_app_ua(name="jd")
    log.info("生成的京东主站App UA:")
    log.info(f"  - App UA: {jd_ua_pair.app[:100]}...") # 打印部分以保持简洁
    log.info(f"  - OkHttp UA: {jd_ua_pair.okhttp}")

if __name__ == "__main__":
    main()
```

### 预期的打印结果

*由于结果是随机的，每次运行的设备信息都会不同，但输出的格式和结构是固定的。*

```
[TIME] | INFO     | [UADemo] : --- 1. 获取一个完全随机的设备信息 ---
[TIME] | INFO     | [UADemo] : 随机抽取的设备品牌: Xiaomi
[TIME] | INFO     | [UADemo] : 设备市场名称: Xiaomi 13 Pro
[TIME] | INFO     | [UADemo] : 具体型号: 2210132C
[TIME] | INFO     | [UADemo] : 开发代号: nuwa
[TIME] | INFO     | [UADemo] : 
--- 2. 根据随机设备信息构建一个通用的浏览器UA ---
[TIME] | INFO     | [UADemo] : 构建的通用UA: Mozilla/5.0 (Linux; Android 13; 2210132C) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Mobile Safari/537.36
[TIME] | INFO     | [UADemo] : 
--- 3. 获取一个指定品牌（华为）的设备信息 ---
[TIME] | INFO     | [UADemo] : 随机华为设备: HUAWEI Mate 60 Pro (ALN-AL00)
[TIME] | INFO     | [UADemo] : 
--- 4. 生成京东App专用User-Agent ---
[TIME] | INFO     | [UADemo] : 生成的京东主站App UA:
[TIME] | INFO     | [UADemo] :   - App UA: jdapp;android;12.3.4;;;M/5.0;appBuild/100500;ef/1;ep/%7B%22hdid%22%3A%22JM9F1ywUPwflvMIpYPok0tt5...
[TIME] | INFO     | [UADemo] :   - OkHttp UA: okhttp/3.12.1;jdmall;android;version/12.3.4;build/100500;
```

---
### **8. 异步文件工具 (`async_file_utils.py`)**

`async_file_utils.py` 提供了一个静态工具类 `FileMethod`，用于执行**异步**且**并发安全**的文件操作。在 `asyncio` 环境中，当有多个协程可能同时写入同一个文件时，使用普通的文件操作会导致数据错乱或文件损坏。本模块通过内置的 `asyncio.Lock` 解决了这个问题，确保所有写操作都是原子性的，是处理配置文件、日志、缓存等场景的理想选择。

## 核心功能 ✨

- **完全异步**: 基于 `aiofiles` 库，所有文件读写操作均为非阻塞，不会影响 `asyncio` 事件循环的性能。
- **并发安全**: 所有的写操作（`write_str`, `write_json`, `write_image`）都被一个全局锁保护，可以防止多个协程同时写入文件造成的数据损坏。
- **智能JSON处理**: `write_json` 方法功能强大，可以自动判断是新建文件、更新现有字典，还是向现有列表追加数据。
- **易于使用**: 所有方法均为类方法，无需实例化，通过 `FileMethod.method_name()`直接调用。
- **多格式支持**: 支持文本 (`str`)、JSON (`dict`/`list`) 和二进制（如图片）文件的读写。

## 方法详解

### `async def read_str(cls, file_name: str, mode: str = "r") -> str`
- **功能**: 异步读取一个文本文件的全部内容。
- **参数**:
  - `file_name` (`str`): 要读取的文件路径。
  - `mode` (`str`, optional): 文件打开模式，默认为 `'r'` (只读)。

### `async def write_str(cls, file_name: str, data: str, mode: str = "a")`
- **功能**: **[并发安全]** 异步地将字符串写入文本文件。
- **参数**:
  - `file_name` (`str`): 要写入的文件路径。
  - `data` (`str`): 要写入的字符串内容。
  - `mode` (`str`, optional): 文件打开模式，默认为 `'a'` (追加)。可使用 `'w'` (覆盖写)。

### `async def read_json(cls, file_name: str) -> Union[Dict, List]`
- **功能**: 异步读取并解析一个JSON文件。
- **返回**: 一个Python字典或列表。

### `async def write_json(cls, file_name: str, updata: Union[Dict, List, str], newBuild: bool = False)`
- **功能**: **[并发安全]** 以智能方式异步写入JSON文件。
- **参数**:
  - `file_name` (`str`): 要写入的文件路径。
  - `updata` (`Union[Dict, List, str]`): 要写入或更新的数据。
  - `newBuild` (`bool`, optional):
    - `True`: 忽略原文件内容，直接用 `updata` 的内容创建或覆盖文件。
    - `False` (默认):
      - 如果原文件是JSON字典，则用 `updata` (必须是字典) 更新它。
      - 如果原文件是JSON列表，则将 `updata` (可以是单个元素或列表) 追加到原列表末尾。

### `async def write_image(cls, file_name: str, image: bytes)`
- **功能**: **[并发安全]** 异步地将二进制数据（如图片内容）写入文件。
- **参数**:
  - `file_name` (`str`): 要保存的文件路径。
  - `image` (`bytes`): 图片的二进制数据。

---

## 实战测试 Demo

下面的Demo将完整地展示 `FileMethod` 的所有核心功能，包括JSON的智能写入和并发安全测试。

```python
# async_file_demo.py
import asyncio
import os
import json
from utils.async_file_utils import FileMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("FileDemo")
JSON_FILE = "demo_data.json"
TEXT_FILE = "concurrent_log.txt"

async def json_operations_demo():
    log.info("--- Demo 1: 演示智能JSON写入操作 ---")
    
    # 1. 使用 newBuild=True 创建一个新文件
    initial_data = {"name": "Alice", "points": 100}
    await FileMethod.write_json(JSON_FILE, initial_data, newBuild=True)
    content_after_create = await FileMethod.read_str(JSON_FILE)
    log.info(f"创建JSON文件后内容:\n{content_after_create}")
    
    # 2. 更新现有JSON字典
    update_data = {"points": 120, "status": "active"}
    await FileMethod.write_json(JSON_FILE, update_data)
    content_after_update = await FileMethod.read_str(JSON_FILE)
    log.info(f"更新字典后内容:\n{content_after_update}")
    
    # 3. 演示对列表的写入
    initial_list = [{"id": 1}]
    await FileMethod.write_json(JSON_FILE, initial_list, newBuild=True) # 先覆盖成列表
    # 追加单个元素
    await FileMethod.write_json(JSON_FILE, {"id": 2})
    # 追加一个列表
    await FileMethod.write_json(JSON_FILE, [{"id": 3}, {"id": 4}])
    content_after_append = await FileMethod.read_str(JSON_FILE)
    log.info(f"追加列表后内容:\n{content_after_append}")


async def concurrent_write_demo():
    log.info("\n--- Demo 2: 演示并发写入安全性 ---")
    
    # a. 定义一个简单的写入任务
    async def write_task(task_id: int):
        await FileMethod.write_str(TEXT_FILE, f"任务 {task_id} 写入成功\n", mode='a')

    # b. 并发地启动10个写入任务
    log.info(f"准备并发执行10个任务，向 '{TEXT_FILE}' 文件中写入内容...")
    tasks = [write_task(i) for i in range(1, 11)]
    await asyncio.gather(*tasks)
    log.info("所有并发写入任务已完成。")

    # c. 读取结果并验证
    final_content = await FileMethod.read_str(TEXT_FILE)
    lines = final_content.strip().split('\n')
    log.info(f"最终文件内容行数: {len(lines)}")
    log.info("由于锁的存在，所有10行内容都应被完整写入，不会出现数据交错或丢失。")


async def main():
    # 清理上次运行的测试文件
    if os.path.exists(JSON_FILE): os.remove(JSON_FILE)
    if os.path.exists(TEXT_FILE): os.remove(TEXT_FILE)

    await json_operations_demo()
    await concurrent_write_demo()
    
    # 再次清理测试文件
    if os.path.exists(JSON_FILE): os.remove(JSON_FILE)
    if os.path.exists(TEXT_FILE): os.remove(TEXT_FILE)


if __name__ == "__main__":
    asyncio.run(main())
```

### 预期的打印结果

```
[TIME] | INFO     | [FileDemo] : --- Demo 1: 演示智能JSON写入操作 ---
[TIME] | INFO     | [FileDemo] : 创建JSON文件后内容:
{
    "name": "Alice",
    "points": 100
}
[TIME] | INFO     | [FileDemo] : 更新字典后内容:
{
    "name": "Alice",
    "points": 120,
    "status": "active"
}
[TIME] | INFO     | [FileDemo] : 追加列表后内容:
[
    {
        "id": 1
    },
    {
        "id": 2
    },
    {
        "id": 3
    },
    {
        "id": 4
    }
]
[TIME] | INFO     | [FileDemo] : 
--- Demo 2: 演示并发写入安全性 ---
[TIME] | INFO     | [FileDemo] : 准备并发执行10个任务，向 'concurrent_log.txt' 文件中写入内容...
[TIME] | INFO     | [FileDemo] : 所有并发写入任务已完成。
[TIME] | INFO     | [FileDemo] : 最终文件内容行数: 10
[TIME] | INFO     | [FileDemo] : 由于锁的存在，所有10行内容都应被完整写入，不会出现数据交错或丢失。
```
*注意：在`concurrent_log.txt`文件中，10行内容的顺序可能是随机的，但每一行本身都是完整且独立的，这证明了并发锁的有效性。*

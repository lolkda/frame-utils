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

### **1. 青龙面板 API (`openApi.py`)**

`openApi.py` 模块提供了与青龙面板进行交互的全部能力，核心是 `openApiCommonMethod` (推荐) 和 `openApiMethod` (底层) 两个类。

#### **核心用法**
1.  **导入**: `from utils.openApi import openApiCommonMethod`。
2.  **实例化**: `ql_api = openApiCommonMethod()`。
3.  **指定面板**: `ql_api.openApi_Use = "PREFIX"`。这里的 `"PREFIX"` 必须与您在 `config.sh` 中配置的 `OPENAPI_PREFIX_...` 前缀一致 (如 "JD", "ELM")。
4.  **调用方法**: `await ql_api.some_method()`。

---
#### **`openApiCommonMethod` (高级封装 - 推荐)**

此类提供了最常用、最便捷的方法，隐藏了底层的API细节，是日常使用的首选。

##### **方法详解**

- **`async def get_cookie(self, ContainerName: str, name: str) -> List[str]`**
  - **功能**: 快速获取指定面板中某个环境变量的所有值。
  - **参数**:
    - `ContainerName`: 面板前缀，如 `"JD"`。
    - `name`: 环境变量名，如 `"JD_COOKIE"`。
  - **返回**: 一个包含所有同名环境变量值的字符串列表 `List[str]`。

- **`async def search_envs(self, keyword: str) -> List[Dict]`**
  - **功能**: 根据关键词模糊搜索环境变量。
  - **参数**: `keyword` (str) - 搜索的关键词。
  - **返回**: 包含环境变量对象字典的列表 `List[Dict]`。例如:
    ```json
    [{
      "id": 123,
      "value": "pt_key=...;",
      "name": "JD_COOKIE",
      "remarks": "用户1",
      "status": 0,
      "timestamp": "2025-07-10T02:30:00.000Z"
    }]
    ```

- **`async def update_envs(self, name: str, value: str, remarks: str, add_env: bool, keyword: str) -> None`**
  - **功能**: 更新或新增一个环境变量。它会先通过 `keyword` 搜索，如果找到就更新，找不到且 `add_env=True` 则新增。
  - **参数**:
    - `name` (str): 环境变量的名称。
    - `value` (str): 环境变量的值。
    - `remarks` (str, optional): 备注。
    - `add_env` (bool, optional): 如果找不到是否新增，默认为`False`。
    - `keyword` (str): 用于定位要更新的环境变量的关键词。

- **`async def disable_envs(self, id: Union[int, List[int]]) -> None`**
  - **功能**: 禁用一个或多个环境变量。
  - **参数**: `id` - 单个环境变量ID (`int`) 或 ID列表 (`List[int]`)。

- **`async def EnableEnvs(self, id: Union[int, List[int]]) -> None`**
  - **功能**: 启用一个或多个环境变量。
  - **参数**: `id` - 单个环境变量ID (`int`) 或 ID列表 (`List[int]`)。

- **`async def search_task(self, keyword: str) -> Dict`**
  - **功能**: 根据关键词搜索定时任务。
  - **返回**: `Dict` - 包含任务列表的字典，任务数据在 `data['data']` 中。

- **`async def run_crons_task(self, id: Union[str, List[str]]) -> None`**
  - **功能**: 运行一个或多个定时任务。
  - **参数**: `id` - 单个任务ID (`str`) 或 ID列表 (`List[str]`)。

##### **实战测试 Demo**

这个Demo将演示一套完整的操作流程：新增 -> 查询 -> 更新 -> 禁用 -> 启用 -> 删除，以及任务的查询和执行。

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
        name=var_name,
        value="initial_value_123",
        remarks="这是一个API测试变量",
        add_env=True, 
        keyword=var_name
    )
    log.info(f"变量 '{var_name}' 已创建或更新。")

    log.info(f"\n--- 2. 搜索并确认变量 ---")
    envs = await ql_api.search_envs(var_name)
    if not envs:
        log.error("致命错误：未能找到刚刚创建的变量。")
        return
    
    env_id = envs[0]['id']
    log.info(f"找到变量，ID: {env_id}, 值为: '{envs[0]['value']}'")

    log.info(f"\n--- 3. 禁用与启用 ---")
    await ql_api.disable_envs(env_id)
    log.info(f"变量 ID:{env_id} 已禁用。")
    await asyncio.sleep(1) 
    await ql_api.EnableEnvs(env_id)
    log.info(f"变量 ID:{env_id} 已重新启用。")
    
    log.info(f"\n--- 4. 运行一个定时任务 ---")
    task_keyword = "签到" # 请替换为您面板中真实存在的任务关键词
    tasks_res = await ql_api.search_task(task_keyword)
    if not tasks_res or not tasks_res.get('data'):
        log.warning(f"未找到包含 '{task_keyword}' 的任务，任务操作跳过。")
    else:
        task = tasks_res['data'][0]
        log.info(f"找到任务: '{task['name']}' (ID: {task['id']})，准备运行...")
        await ql_api.run_crons_task(task['id'])
        log.info("任务已触发运行。")

    log.info(f"\n--- 5. 清理测试变量 ---")
    # 使用底层接口删除变量
    await ql_api.delete_openApi('envs', [env_id])
    log.info(f"测试变量 ID:{env_id} 已被删除。")

if __name__ == "__main__":
    asyncio.run(main())
```

**预期打印结果**:
```
[TIME] | INFO     | [QL_Demo] : --- 1. 新增/更新环境变量 'MY_API_TEST_VAR' ---
[TIME] | INFO     | [QL_Demo] : 变量 'MY_API_TEST_VAR' 已创建或更新。
[TIME] | INFO     | [QL_Demo] : 
--- 2. 搜索并确认变量 ---
[TIME] | INFO     | [QL_Demo] : 找到变量，ID: 123, 值为: 'initial_value_123'
[TIME] | INFO     | [QL_Demo] : 
--- 3. 禁用与启用 ---
[TIME] | INFO     | [QL_Demo] : 变量 ID:123 已禁用。
[TIME] | INFO     | [QL_Demo] : 变量 ID:123 已重新启用。
[TIME] | INFO     | [QL_Demo] : 
--- 4. 运行一个定时任务 ---
[TIME] | INFO     | [QL_Demo] : 找到任务: '京东签到' (ID: abc-123)，准备运行...
[TIME] | INFO     | [QL_Demo] : 任务已触发运行。
[TIME] | INFO     | [QL_Demo] : 
--- 5. 清理测试变量 ---
[TIME] | INFO     | [QL_Demo] : 测试变量 ID:123 已被删除。
```

---
#### **`openApiMethod` (底层接口 - 进阶使用)**

此类提供了对青龙 OpenAPI 端点的直接 `GET/PUT/POST/DELETE` 访问，适合需要高度自定义请求或调用 `openApiCommonMethod` 未封装接口的场景。

- **`get_openApi(query, params=None)`**: 执行 `GET` 请求。
- **`put_openApi(query, json)`**: 执行 `PUT` 请求，通常用于更新或执行动作。
- **`post_openApi(query, json)`**: 执行 `POST` 请求，通常用于创建新资源。
- **`delete_openApi(query, json)`**: 执行 `DELETE` 请求，用于删除资源。

##### **测试Demo**
```python
# openapi_low_level_demo.py
import asyncio
from utils.openApi import openApiMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("QL_LowLevel_Demo")

async def main():
    ql_api = openApiMethod()
    ql_api.openApi_Use = "JD"

    log.info("--- 1. POST: 新增一个环境变量 ---")
    new_env_data = [{"name": "LOW_LEVEL_VAR", "value": "test", "remarks": "底层接口测试"}]
    await ql_api.post_openApi('envs', new_env_data)
    log.info("环境变量已新增。")

    log.info("\n--- 2. GET: 搜索该变量 ---")
    envs = await ql_api.get_openApi('envs', params={'searchValue': 'LOW_LEVEL_VAR'})
    env_to_operate = envs['data'][0] if envs and envs.get('data') else None
    if not env_to_operate:
        log.error("未能找到变量。")
        return
    log.info(f"找到变量，ID: {env_to_operate['id']}")
    
    log.info("\n--- 3. PUT: 更新该变量 ---")
    env_to_operate['value'] = 'updated_by_put'
    await ql_api.put_openApi('envs', env_to_operate)
    log.info("环境变量已更新。")

    log.info("\n--- 4. DELETE: 删除该变量 ---")
    await ql_api.delete_openApi('envs', [env_to_operate['id']])
    log.info("环境变量已删除。")

if __name__ == "__main__":
    asyncio.run(main())
```

**预期打印结果**:
```
[TIME] | INFO     | [QL_LowLevel_Demo] : --- 1. POST: 新增一个环境变量 ---
[TIME] | INFO     | [QL_LowLevel_Demo] : 环境变量已新增。
[TIME] | INFO     | [QL_LowLevel_Demo] :
--- 2. GET: 搜索该变量 ---
[TIME] | INFO     | [QL_LowLevel_Demo] : 找到变量，ID: 124
[TIME] | INFO     | [QL_LowLevel_Demo] :
--- 3. PUT: 更新该变量 ---
[TIME] | INFO     | [QL_LowLevel_Demo] : 环境变量已更新。
[TIME] | INFO     | [QL_LowLevel_Demo] :
--- 4. DELETE: 删除该变量 ---
[TIME] | INFO     | [QL_LowLevel_Demo] : 环境变量已删除。
```

---
### **2. 环境配置读取 (`env_utils.py`)**

`EnvMethod` 类是读取项目配置的核心，提供了比 `os.getenv` 更强大的功能。

#### **方法详解**

- **`readEnv(key, default=None, codeInt=False, codeList=False)`**
  - **功能**: 智能读取环境变量。如果值是 `true` 或 `false`，会自动转为布尔型；如果值是 `"[...]"`, `"{...}"` 格式，会尝试用 `json.loads` 解析；如果包含 `|`，会自动分割为列表。
  - **参数**:
    - `key` (str): 环境变量名。
    - `default` (any): 变量不存在时返回的默认值。
    - `codeInt` (bool): `True` 时，强制将结果（或列表中的每一项）转换为整数。
    - `codeList` (bool): `True` 时，即使不包含 `|`，也强制将结果分割为列表。

- **`load_config_from_env(obj_self, config_obj, err_exit=True)`**
  - **功能**: 自动化的配置加载器。它会检查 `config_obj` 的类型提示 (`__annotations__`) 和 `env_map`，自动读取并转换环境变量，然后设置到 `obj_self` 对象上。
  - **参数**:
    - `obj_self`: 要被设置属性的目标对象（通常是 `self`）。
    - `config_obj`: 一个定义了配置结构和默认值的类实例。

#### **测试Demo**
```python
# env_demo.py
import os
from typing import List
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

# --- Demo Part 1: 测试 readEnv ---
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

# --- Demo Part 2: 测试 load_config_from_env ---
log.info("\n--- 测试 load_config_from_env ---")

# 定义一个配置模板类
class MyScriptConfig:
    env_map = {
        'str_setting': 'DEMO_STR',
        'int_setting': 'DEMO_INT',
        'some_items': 'DEMO_LIST_PIPE'
    }
    # 定义属性和类型提示，以及默认值
    some_string: str = "default_string"
    some_number: int = 0
    some_items: List[str] = []

# 在你的主程序类中使用
class MyScript:
    def __init__(self):
        # 将配置加载到 self 的属性上
        EnvMethod.load_config_from_env(self, MyScriptConfig())

app = MyScript()
log.info(f"加载后 app.some_string = '{app.some_string}'")
log.info(f"加载后 app.some_number = {app.some_number}")
log.info(f"加载后 app.some_items = {app.some_items}")
```

**预期打印结果**:
```
[TIME] | INFO     | [EnvDemo] : --- 测试 readEnv ---
[TIME] | INFO     | [EnvDemo] : 读取字符串: 'hello world' (类型: <class 'str'>)
[TIME] | INFO     | [EnvDemo] : 读取整数: 123 (类型: <class 'int'>)
[TIME] | INFO     | [EnvDemo] : 读取管道列表: ['a', 'b', 'c'] (类型: <class 'list'>)
[TIME] | INFO     | [EnvDemo] : 读取JSON列表: [1, 2, 3] (类型: <class 'list'>)
[TIME] | INFO     | [EnvDemo] : 读取布尔值: True (类型: <class 'bool'>)
[TIME] | INFO     | [EnvDemo] : 读取默认值: '我是默认值' (类型: <class 'str'>)
[TIME] | INFO     | [EnvDemo] : 
--- 测试 load_config_from_env ---
[TIME] | INFO     | [EnvDemo] : 加载后 app.some_string = 'hello world'
[TIME] | INFO     | [EnvDemo] : 加载后 app.some_number = 123
[TIME] | INFO     | [EnvDemo] : 加载后 app.some_items = ['a', 'b', 'c']
```
---
### **3. 通知发送 (`sendNotify.py`) 与 插件开发**

`sendNotify.py` 模块提供了一个可扩展的通知发送框架。

#### **核心用法**
`SendMethod` 类会自动扫描 `function/push_plugins/` 目录下的所有插件，并加载已启用的插件。

- **`SendParam(title, content, uids=None)`**: 用于封装通知内容的标准数据类。
- **`send_all(param)`**: 向所有启用的渠道广播消息。
- **`send_to(sender_class, param)`**: 向指定的单个渠道发送消息。

#### **测试Demo**
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

**预期打印结果**:
```
[TIME] | INFO     | [NotifyDemo] : 准备向所有启用的渠道发送通知...
[TIME] | INFO     | [sendNotify] : 开始向 1 个启用的推送器（全部）发送消息: 自动化任务报告
[TIME] | INFO     | [sendNotify] | [PushDeerSender] : 开始向指定推送器 [PushDeerSender] 发送消息: 自动化任务报告
[TIME] | INFO     | [sendNotify] | [PushDeerSender] : PushDeer 推送成功！
[TIME] | INFO     | [sendNotify] | [PushDeerSender] : 指定推送任务 [PushDeerSender] 执行成功。
```
---
#### **开发者指南：如何添加新的推送插件 (保姆级教程)**

本工具库的通知系统是插件化的，这意味您可以非常轻松地添加任何您想使用的推送服务。下面我们以添加一个虚构的 **PushDeer** 推送服务为例，一步步教您如何操作。

##### **第一步: 创建插件文件**
在 `function/push_plugins/` 目录下，创建一个新的Python文件，命名为您推送服务的名字，例如 `pushdeer.py`。

##### **第二步: 编写完整类代码**
打开 `pushdeer.py` 文件，将以下完整代码复制进去。这段代码包含了所有必需的逻辑。

```python
# function/push_plugins/pushdeer.py
import json
from utils.sendNotify import BaseSender
from utils.env_utils import EnvMethod

class PushDeerSender(BaseSender):
    """
    PushDeer 推送插件
    官方文档: [https://www.pushdeer.com/](https://www.pushdeer.com/)
    """
    def __init__(self):
        """
        初始化 PushDeer 推送器。
        - 调用父类构造函数 (super().__init__()) 来获取 self.req (HTTP请求器) 和 self.log (日志记录器)。
        - 从环境变量读取 PUSHDEER_ISOPEN 和 PUSHDEER_KEY。
        """
        super().__init__()
        
        # 从环境变量读取配置
        self.is_open = EnvMethod.readEnv("PUSHDEER_ISOPEN", "false").lower() == "true"
        self.key = EnvMethod.readEnv("PUSHDEER_KEY")
        
        # 定义API地址
        self.api_url = "[https://api2.pushdeer.com/message/push](https://api2.pushdeer.com/message/push)"

    def is_enabled(self) -> bool:
        """
        判断此推送器是否已在环境变量中正确配置并启用。
        只有当 PUSHDEER_ISOPEN 为 "true" 且 PUSHDEER_KEY 有值时，才算启用。
        """
        return self.is_open and bool(self.key)

    async def send(self, title: str, content: str, **kwargs) -> bool:
        """
        实现具体的发送逻辑。
        """
        # 如果未启用，直接返回失败，不执行任何操作
        if not self.is_enabled():
            return False
            
        # 1. 构造请求体 (Payload)
        # PushDeer的API需要 'pushkey', 'text', 'desp'
        # 我们将标题作为 text，内容作为 desp (支持Markdown)
        payload = {
            "pushkey": self.key,
            "text": title,
            "desp": content.replace('\n', '\n\n'), # PushDeer使用markdown，换两行才是真正的换行
            "type": "markdown" # 指定类型为markdown
        }
        
        # 2. 构造完整的HTTP请求参数
        params = {
            "method": "POST",
            "url": self.api_url,
            "data": payload # PushDeer 使用 form-data，所以用 'data' 而不是 'json'
        }
        
        # 3. 调用HTTP客户端发送请求
        response = await self.req.async_curl_requests(params, "PushDeer")
        
        # 4. 判断结果并返回布尔值
        if response.status == 200:
            try:
                res_json = json.loads(response.text)
                # 根据PushDeer的返回格式判断是否成功
                if res_json.get("code") == 0:
                    self.log.info("PushDeer 推送成功！")
                    return True
            except Exception as e:
                self.log.error(f"PushDeer 推送成功，但解析返回JSON时出错: {e}", exit=False)
                return False
        
        # 如果HTTP状态码不为200或API返回错误码
        self.log.error(f"PushDeer 推送失败: {response.text}", exit=False)
        return False
```

##### **第三步: 添加配置到 `config.sh`**
打开 `/env/config.sh` 文件，在末尾添加PushDeer的配置。

```shell
# --- 推送通知插件配置 (PushDeer示例) ---
export PUSHDEER_ISOPEN="true"
export PUSHDEER_KEY="PDU123456789xxxx" # 替换成你自己的PushDeer Key
```

##### **完成!**
至此，您已成功添加了一个全新的推送插件。当您在主脚本中调用 `SendMethod().send_all(...)` 时，程序会自动发现并执行您的 `PushDeerSender`。

---

## 许可证

本项目采用 [MIT](https://opensource.org/licenses/MIT) 许可证。

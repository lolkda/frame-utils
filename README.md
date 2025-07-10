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

以下是各核心模块的详细使用方法和代码示例。

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
  - **返回**: 一个包含所有同名环境变量值的字符串列表。

- **`async def search_envs(self, keyword: str) -> List[Dict]`**
  - **功能**: 根据关键词模糊搜索环境变量。
  - **参数**: `keyword` - 搜索的关键词。
  - **返回**: 包含环境变量对象字典的列表。例如:
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
    - `name`, `value`, `remarks`: 环境变量的名称、值和备注。
    - `add_env`: (可选, `bool`) 如果找不到是否新增，默认为`False`。
    - `keyword`: 用于定位要更新的环境变量的关键词。

- **`async def disable_envs(self, id: Union[int, List[int]]) -> None`**
  - **功能**: 禁用一个或多个环境变量。
  - **参数**: `id` - 单个环境变量ID (`int`) 或 ID列表 (`List[int]`)。

- **`async def EnableEnvs(self, id: Union[int, List[int]]) -> None`**
  - **功能**: 启用一个或多个环境变量。
  - **参数**: `id` - 单个环境变量ID (`int`) 或 ID列表 (`List[int]`)。

- **`async def search_task(self, keyword: str) -> Dict`**
  - **功能**: 根据关键词搜索定时任务。
  - **返回**: 包含任务列表的字典，任务数据在 `data['data']` 中。

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
    # 验证状态 (实际使用中可以加延迟或轮询)
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

---
#### **`openApiMethod` (底层接口 - 进阶使用)**

此类提供了对青龙 OpenAPI 端点的直接 `GET/PUT/POST/DELETE` 访问，适合需要高度自定义请求或调用 `openApiCommonMethod` 未封装接口的场景。

##### **方法详解**

- **`async def get_openApi(self, query: str, params: Dict = None) -> Dict`**: 执行 `GET` 请求。
  - `query`: API路径，如 `'envs'`, `'crons'`, `'configs/config.sh'`。
  - `params`: (可选) URL查询参数字典。
- **`async def put_openApi(self, query: str, json: Union[List, Dict]) -> Dict`**: 执行 `PUT` 请求。
  - `query`: API路径，如 `'envs/enable'`, `'crons/run'`, `'envs'`。
  - `json`: 发送的JSON body。
- **`async def post_openApi(self, query: str, json: Union[List, Dict]) -> Dict`**: 执行 `POST` 请求。
  - `query`: API路径，如 `'envs'`, `'configs/save'`。
  - `json`: 发送的JSON body。
- **`async def delete_openApi(self, query: str, json: List[int]) -> Dict`**: 执行 `DELETE` 请求。
  - `query`: API路径，如 `'envs'`。
  - `json`: 包含要删除资源ID的列表。

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

### **2. 开发者: 如何添加新的推送插件 (保姆级教程)**

本工具库的通知系统是插件化的，这意味您可以非常轻松地添加任何您想使用的推送服务。下面我们以添加一个虚构的 **PushDeer** 推送服务为例，一步步教您如何操作。

#### **第一步: 创建插件文件**
在 `function/push_plugins/` 目录下，创建一个新的Python文件，命名为您推送服务的名字，例如 `pushdeer.py`。

#### **第二步: 编写类结构**
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
    # 后面我们将在这里填充代码
    pass
```

#### **第三步: 实现 `__init__` 构造方法**
在 `__init__` 中，我们要调用父类的构造方法，并从 `config.sh` 读取本插件所需的所有配置。

```python
# ... (imports) ...

class PushDeerSender(BaseSender):
    def __init__(self):
        # 1. 必须调用父类的构造方法
        super().__init__()
        
        # 2. 从环境变量读取本插件的配置
        self.is_open = EnvMethod.readEnv("PUSHDEER_ISOPEN", "false").lower() == "true"
        self.key = EnvMethod.readEnv("PUSHDEER_KEY")
        
        # 3. (可选) 您也可以在这里定义API地址等固定信息
        self.api_url = "[https://api2.pushdeer.com/message/push](https://api2.pushdeer.com/message/push)"
```

#### **第四步: 实现 `is_enabled` 方法**
此方法用于告诉主程序，当前插件是否配置正确并已启用。

```python
# ... (imports) ...

class PushDeerSender(BaseSender):
    def __init__(self):
        # ... (同上) ...
    
    def is_enabled(self) -> bool:
        """
        判断此推送器是否已在环境变量中启用。
        """
        # 只有当 PUSHDEER_ISOPEN 为 true 且 PUSHDEER_KEY 有值时，才算启用
        return self.is_open and bool(self.key)
```

#### **第五步: 实现 `send` 方法**
这是最核心的方法，负责执行实际的推送逻辑。

```python
# ... (imports) ...

class PushDeerSender(BaseSender):
    # ... (__init__ 和 is_enabled 方法) ...

    async def send(self, title: str, content: str, **kwargs) -> bool:
        """
        发送消息的统一接口。
        """
        if not self.is_enabled():
            return False # 如果未启用，直接返回失败
            
        # 1. 构造请求参数
        # PushDeer的API需要 'pushkey', 'text', 'desp'
        params = {
            "method": "POST",
            "url": self.api_url,
            "data": {
                "pushkey": self.key,
                "text": title,
                "desp": content
            }
        }
        
        # 2. 调用HTTP客户端发送请求
        # `self.req` 是从 BaseSender 继承而来的 AsyncRequestManager 实例
        # `self.log` 是从 BaseSender 继承而来的 PrintMethodClass 实例
        response = await self.req.async_curl_requests(params, "PushDeer")
        
        # 3. 判断结果并返回布尔值
        if response.status == 200:
            res_json = response.json()
            if res_json.get("code") == 0:
                self.log.info("PushDeer 推送成功！")
                return True
        
        self.log.error(f"PushDeer 推送失败: {response.text}")
        return False
```

#### **第六步: 添加配置到 `config.sh`**
打开 `/env/config.sh` 文件，在末尾添加PushDeer的配置。

```shell
# --- 推送通知插件配置 (PushDeer示例) ---
export PUSHDEER_ISOPEN="true"
export PUSHDEER_KEY="PDU123456789xxxx" # 替换成你自己的PushDeer Key
```

#### **完成!**
至此，您已成功添加了一个全新的推送插件。当您在主脚本中调用 `SendMethod().send_all(...)` 时，程序会自动发现并执行您的 `PushDeerSender`。

---
### **3. 日志工具 (`logging_utils.py`)**

`logging_utils.py` 提供了一个强大且灵活的日志记录工具 `PrintMethodClass`。

#### **测试Demo**
```python
# logging_demo.py
import asyncio
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("UserProcessor", auto_reset=False)

async def process_user(user_id, use_proxy):
    log.set("UserID", user_id)
    if use_proxy:
        log.set("Network", "Proxy")
    log.info("开始处理该用户...")
    await asyncio.sleep(0.5) 
    log.info("用户处理完毕。")
    log.reset()

async def main():
    log.info("脚本开始运行...")
    await process_user("user-001", use_proxy=False)
    await process_user("user-007", use_proxy=True)
    log.info("脚本运行结束。")

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **4. 异步并发控制器 (`concurrency_utils.py`)**

`RunMethod` 类用于以指定的并发数批量执行异步任务。

#### **测试Demo**
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
    tasks_to_run = [
        {'id': 1, 'delay': 2}, {'id': 2, 'delay': 1}, {'id': 3, 'delay': 3},
        {'id': 4, 'delay': 1}, {'id': 5, 'delay': 2}, {'id': 6, 'delay': 1.5},
    ]
    conc_params = ReqConcParam(
        func=worker_task, task=tasks_to_run, thread=3, wait=2, extra_param="FixedValue"
    )
    async for batch_result in RunMethod.ReqConcRun(conc_params):
        log.info(f"--- 一批任务执行完毕 ---")
        for res in batch_result:
            log.info(f"结果: {res.result}")

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **5. 异步进程管理器 (`script_executor.py`)**

`ProcessManager` 类能够以非阻塞的方式启动、监控和管理外部脚本或系统命令。

#### **测试Demo**
```python
# executor_demo.py
import asyncio
from utils.script_executor import ProcessManager
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("ExecutorDemo")

with open("my_test_script.py", "w") as f:
    f.write('import time\nprint("脚本运行中...")\ntime.sleep(2)\nprint("脚本结束")')

async def main():
    manager = ProcessManager()
    pid = await manager.run_command(['python', 'my_test_script.py'])
    log.info(f"脚本已启动，PID: {pid}")
    result_info = await manager.wait_for_process(pid)
    log.info(f"命令输出:\n{result_info.stdout.strip()}")

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **6. 异步安全文件读写 (`async_file_utils.py`)**

`FileMethod` 类提供了一系列带锁的异步文件操作方法。

#### **测试Demo**
```python
# file_demo.py
import asyncio
from utils.async_file_utils import FileMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("FileDemo")
JSON_FILE = "demo_data.json"

async def main():
    await FileMethod.write_json(JSON_FILE, {"count": 1}, newBuild=True)
    log.info(f"已创建JSON文件: {await FileMethod.read_str(JSON_FILE)}")
    
    await FileMethod.write_json(JSON_FILE, {"status": "active"})
    log.info(f"更新后: {await FileMethod.read_str(JSON_FILE)}")

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **7. 环境配置读取 (`env_utils.py`)**

`EnvMethod` 类是读取项目配置的核心。

#### **测试Demo**
```python
# env_demo.py
import os
from utils.env_utils import EnvMethod
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("EnvDemo")
os.environ['DEMO_LIST'] = "a|b|c"
list_val = EnvMethod.readEnv("DEMO_LIST", [])
log.info(f"读取到列表: {list_val} (类型: {type(list_val)})")
```

---
### **8. HTTP 客户端 (`http_client.py`)**

`AsyncRequestManager` 是一个功能强大的异步HTTP请求器。

#### **测试Demo**
```python
# http_demo.py
import asyncio
from utils.http_client import AsyncRequestManager
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("HttpDemo")
async def main():
    req = AsyncRequestManager()
    params = {"method": "GET", "url": "[https://api.ipify.org?format=json](https://api.ipify.org?format=json)"}
    response = await req.async_curl_requests(params, "GetMyIP")
    log.info(f"我的IP是: {response.text}")

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **9. 任务调度器 (`time_scheduler.py`)**

`ServerTimeScheduler` 可用于需要精确时间的任务场景。

#### **测试Demo**
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
    target_time = datetime.now() + timedelta(seconds=3)
    log.info(f"准备等待到 {target_time.strftime('%H:%M:%S')}")
    await scheduler.wait_until(
        hour=target_time.hour, minute=target_time.minute, second=target_time.second
    )
    await my_timed_task()

if __name__ == "__main__":
    asyncio.run(main())
```

---
### **10. User-Agent 生成器 (`user_agent_generator.py`)**

轻松生成各种真实的移动设备User-Agent。

#### **测试Demo**
```python
# ua_demo.py
from utils.user_agent_generator import PhoneModel
from utils.logging_utils import PrintMethodClass

log = PrintMethodClass("UADemo")
random_phone = PhoneModel.get_phone_models(brand="Xiaomi")
log.info(f"随机小米设备: {random_phone.name} ({random_phone.model_number})")
```

---
### **11. 通知发送 (`sendNotify.py`)**

`SendMethod` 负责加载所有启用的推送插件并发送消息。

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
        log.warning("未配置或启用任何推送器。")
        return
    params = SendParam(
        title="任务报告", 
        content=f"所有任务已于 {datetime.now()} 完成。"
    )
    await notify.send_all(params)
    log.info("通知已发送。")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 许可证

本项目采用 [MIT](https://opensource.org/licenses/MIT) 许可证。

# Python 高性能异步自动化工具库

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/)

这是一个专为自动化任务设计的Python工具库，具备高性能、异步、可扩展的特点。它内置了强大的HTTP客户端、代理管理、任务调度、设备UA生成等基础功能，并深度集成了 **京东(JD.com) API** 和 **青龙(Qinglong)面板** 的交互能力，是构建复杂自动化流程的理想选择。

## 核心功能 ✨

- **高性能HTTP客户端**: 基于 `curl_cffi` 实现，支持HTTP/2、TLS指纹伪装，具备自动重试和灵活的代理管理功能。
- **京东API深度集成**: 内置了京东h5st签名算法的调用接口，封装了大量的常用京东活动API，简化了与京东服务端的交互。
- **青龙面板无缝对接**: 强大的青龙OpenAPI客户端，可轻松通过代码管理面板中的环境变量、定时任务、配置文件等。
- **灵活的任务调度器**: 支持与服务器时间同步，可实现高精度的定时、延时、周期性任务调度。
- **可扩展的通知系统**: 采用插件化设计，可以轻松集成多种推送渠道（如Telegram, Pushplus等），用于任务结果的实时通知。
- **强大的环境与配置管理**: 简化了环境变量的读取和类型转换，支持从 `config.sh` 文件加载配置。
- **丰富的User-Agent生成器**: 可生成覆盖苹果、华为、小米等主流品牌的上千种真实移动设备User-Agent，增强请求的模拟度。
- **结构化的日志系统**: 提供清晰、可定制的日志输出，支持按脚本和日期生成日志文件，便于调试和追踪。

---

## 项目结构

```
.
├── function/
│   ├── push_plugins/       # 推送插件目录
│   │   └── ...
│   └── jd_api.py           # 封装的京东API
├── utils/
│   ├── env_utils.py        # 环境配置读取工具
│   ├── http_client.py      # 异步HTTP客户端
│   ├── logging_utils.py    # 日志工具
│   ├── openApi.py          # 青龙面板API客户端
│   ├── sendNotify.py       # 推送通知管理器
│   ├── time_scheduler.py   # 异步任务调度器
│   └── user_agent_generator.py # UA生成器
├── env/
│   └── config.sh           # 唯一的项目配置文件
└── main_script.py          # 你的主脚本文件
```

---

## 快速开始 🚀

### 1. 环境准备

确保您的环境中已安装 Python 3.8+。

### 2. 安装依赖

项目依赖于 `curl_cffi` 等第三方库，可以通过以下命令安装：

```bash
pip install curl_cffi
```

### 3. 项目配置 (`/env/config.sh`)

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

### 1. 日志 (`logging_utils.py`)

提供结构化的日志记录功能。

```python
from utils.logging_utils import PrintMethodClass

# 初始化日志记录器，可以传入脚本名
log = PrintMethodClass("MyScript")

# 记录不同级别的日志
log.info("这是一条普通信息。")
log.warning("这是一条警告信息。")
log.error("发生了一个错误！", exit=False) # exit=False表示记录错误后不退出程序

# 在日志中添加临时上下文信息
log.set("UserID", "user123")
log.info("正在处理用户数据。") # 日志会显示 [MyScript] | [user123] : 正在处理用户数据。
log.reset() # 清除临时信息
log.info("处理完毕。") # 日志恢复为 [MyScript] : 处理完毕。
```

### 2. 环境配置读取 (`env_utils.py`)

安全、便捷地从 `config.sh` 读取配置。

```python
from utils.env_utils import EnvMethod

# 读取字符串
proxy_url = EnvMethod.readEnv("PROXY_URL", "default_url")
print(f"代理地址: {proxy_url}")

# 读取并转换为整数
timeout = EnvMethod.readEnv("Req_Timeout", 10, codeInt=True)
print(f"超时时间: {timeout} 秒")

# 读取并转换为列表
# 假设 config.sh 中有: export MY_LIST="item1|item2"
my_list = EnvMethod.readEnv("MY_LIST", [])
print(f"列表内容: {my_list}")
```

### 3. HTTP 客户端 (`http_client.py`)

发送异步网络请求的核心。

```python
import asyncio
from utils.http_client import AsyncRequestManager

async def main():
    req = AsyncRequestManager()

    # 构造请求参数字典
    params = {
        "method": "GET",
        "url": "[https://api.ipify.org?format=json](https://api.ipify.org?format=json)",
        "proxy": True  # 自动使用配置的代理
    }

    # 发送请求
    response = await req.async_curl_requests(params, "GetMyIP")

    if response.status == 200:
        print(f"请求成功: {response.text}")
    else:
        print(f"请求失败，状态码: {response.status}")

# 运行异步主函数
asyncio.run(main())
```

### 4. 青龙面板 API (`openApi.py`)

与青龙面板进行交互，管理环境和任务。

```python
import asyncio
from utils.openApi import openApiCommonMethod

async def main():
    # 使用通用方法类
    ql_api = openApiCommonMethod()
    
    # --- 示例1: 获取并更新环境变量 ---
    # 指定要操作的青龙面板实例 (与config.sh中的前缀对应)
    ql_api.openApi_Use = "JD" 
    
    # 搜索环境变量
    print("正在搜索 JD_COOKIE...")
    envs = await ql_api.search_envs("JD_COOKIE")
    if envs:
        first_env = envs[0]
        print(f"找到环境变量: ID={first_env['id']}, Name={first_env['name']}")
        
        # 更新环境变量 (这里仅为示例，实际值请替换)
        await ql_api.update_envs(
            name="JD_COOKIE", 
            value="pt_key=new_key;pt_pin=new_pin;", 
            remarks="由脚本自动更新",
            keyword="JD_COOKIE" # 使用关键词定位
        )
        print("环境变量已更新。")
    
    # --- 示例2: 运行定时任务 ---
    ql_api.openApi_Use = "ELM" # 切换到另一个面板
    
    print("\n正在搜索 '饿了么' 任务...")
    tasks = await ql_api.search_task("饿了么")
    if tasks and tasks.get('data'):
        first_task = tasks['data'][0]
        task_id = first_task['id']
        print(f"找到任务: ID={task_id}, Name={first_task['name']}")
        
        # 运行此任务
        await ql_api.run_crons_task(task_id)
        print(f"任务 '{first_task['name']}' 已触发运行。")

# 运行异步主函数
asyncio.run(main())
```

### 5. 京东 API (`jd_api.py`)

封装了复杂的签名和API调用逻辑，让你可以专注于业务。

```python
import asyncio
from utils.jd_api import JdApiClient

# 假设你已在青龙或环境变量中设置了名为 JD_COOKIE 的京东Cookie
# cookie = "pt_key=...;pt_pin=...;" 

async def main():
    jd_client = JdApiClient()
    # 从青龙面板获取Cookie
    # cookie_list = await ql_api.get_cookie("JD", "JD_COOKIE") 
    cookie = "你的京东Cookie" # 此处为了演示，直接使用字符串
    
    # 示例：执行店铺关注领豆
    # 假设已知店铺ID和活动ID
    shopId = "1000004123"
    venderId = "1000004123"
    activityId = "6a21648a-861c-4389-9b93-6b71836171f1" # 这是一个示例ID
    
    print(f"正在为店铺 {shopId} 尝试关注领豆...")
    response = await jd_client.get_whx_drawShopGift(
        Cookie=cookie, 
        shopId=shopId, 
        venderId=venderId, 
        activityId=activityId
    )
    
    if response.status == 200:
        result = response.text
        # 根据京东返回的JSON解析结果
        print(f"关注结果: {result}")
    else:
        print(f"请求失败: {response.status}")

# 运行
asyncio.run(main())
```

### 6. 任务调度器 (`time_scheduler.py`)

用于执行时间敏感的任务。

```python
import asyncio
from datetime import datetime
from utils.time_scheduler import ServerTimeScheduler
from utils.jd_api import JdApiClient

# 获取京东服务器时间作为时间同步函数
jd_client = JdApiClient()
time_sync_func = jd_client.Jd_Time

async def my_task():
    print(f"任务执行于: {datetime.now()}")

async def main():
    # 初始化调度器，并传入时间同步函数
    scheduler = ServerTimeScheduler(func=time_sync_func)
    
    # 示例1: 等待到下一个 10:30:00
    print("等待到下一个 10:30:00...")
    await scheduler.wait_until(hour=10, minute=30, second=0)
    await my_task()
    
    # 示例2: 等待到下一个活跃时间段 (例如凌晨2-4点，或早上8点)
    print("\n检查是否在活跃时间段...")
    await scheduler.sleep_until_next_active_period(active_hours=["2-4", "8"])
    print("已到达活跃时段，开始执行任务。")
    await my_task()

# 运行
asyncio.run(main())
```

### 7. 通知发送 (`sendNotify.py`)

用于脚本执行完毕后发送通知。

```python
import asyncio
from utils.sendNotify import SendMethod, SendParam
# from function.push_plugins.telegram import TelegramSender # 假设这是你的一个插件类

async def main():
    notify = SendMethod()
    
    # 构建通知内容
    title = "每日任务报告"
    content_list = [
        "任务A: 成功",
        "任务B: 失败 - 原因: Cookie失效",
        "任务C: 获得 10 京豆"
    ]
    
    # 创建发送参数
    params = SendParam(title=title, content=content_list)
    
    # --- 发送给所有启用的通知渠道 ---
    await notify.send_all(params)
    
    # --- (高级用法) 发送给指定的渠道 ---
    # 假设你知道插件类名，并且它已启用
    # from function.push_plugins.telegram import TelegramSender # 确保导入了插件类
    # await notify.send_to(TelegramSender, params)

# 运行
asyncio.run(main())
```

### 8. User-Agent 生成器 (`user_agent_generator.py`)

为你的请求提供真实的设备指纹。

```python
from utils.user_agent_generator import PhoneModel, JdUserAgentGenerator

# 获取一个随机的安卓设备User-Agent
random_phone = PhoneModel.get_phone_models()
android_version = "13"
ua_string = f"Mozilla/5.0 (Linux; Android {android_version}; {random_phone.model_number}) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Mobile Safari/537.36"
print(f"生成的随机UA: {ua_string}")
print(f"设备信息: 品牌={random_phone.brand}, 名称={random_phone.name}")

# 生成京东App专用的User-Agent
jd_ua_gen = JdUserAgentGenerator(clientVersion="12.3.4", build="100500")
jd_ua = jd_ua_gen.jd_app_ua(name="jd")
print(f"\n生成的京东App UA: {jd_ua.app}")
print(f"生成的京东OkHttp UA: {jd_ua.okhttp}")
```

---
## 开发者: 如何添加新的推送插件

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
        # 调用父类的构造函数
        super().__init__()
        # 读取此插件所需的配置
        self.is_open = EnvMethod.readEnv("MY_PUSHER_ISOPEN", "false").lower() == "true"
        self.api_key = EnvMethod.readEnv("MY_PUSHER_APIKEY")

    def is_enabled(self) -> bool:
        """
        通过环境变量判断此推送器是否启用。
        """
        return self.is_open and bool(self.api_key)

    async def send(self, title: str, content: str, **kwargs) -> bool:
        """
        实现具体的发送逻辑。
        """
        if not self.is_enabled():
            self.log.warning("我的推送器未启用或未配置API Key。")
            return False
            
        url = "[https://api.mypusher.com/push](https://api.mypusher.com/push)"
        payload = {
            "key": self.api_key,
            "title": title,
            "message": content
        }
        
        params = {
            "method": "POST",
            "url": url,
            "json": payload
        }
        
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

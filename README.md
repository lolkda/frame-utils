# 📦 环境变量、异步文件与并发工具库使用文档

---

## 模块目录

| 模块名                | 说明 |
|-----------------------|------|
| `env_utils.py`        | 环境变量操作工具 |
| `async_file_utils.py` | 异步文件读写工具 |
| `concurrency_utils.py`| 并发任务调度工具 |

---

`concurrency_utils.py` 是一个专为异步并发处理设计的工具库，提供了多种并发执行模式，适用于需要高效处理大量异步任务的场景。

## 核心组件

### 1. ReqConcParam 类
用于封装并发执行所需的参数配置。

#### 构造函数
```python
ReqConcParam(func, task, thread, wait=0, **kwargs)
```

#### 参数说明
- `func`: 要执行的函数或函数列表
- `task`: 任务列表或元组
- `thread`: 并发线程数（最大200）
- `wait`: 批次间等待时间（秒）
- `**kwargs`: 额外的关键字参数

#### 使用示例
```python
param = ReqConcParam(
    func=my_async_function,
    task=[task1, task2, task3],
    thread=5,
    wait=1.0,
    timeout=30
)
```

### 2. ReqConcResult 类
封装单个任务的执行结果。

#### 属性
- `result`: 异步响应结果 (AsyncResponse)
- `task`: 对应的任务对象

### 3. RunMethod 类
提供多种并发执行方法的核心类。

## 执行方法

### 1. ConcRun - 基础并发执行
适用于执行相同函数的多个并发任务。

```python
results = await RunMethod.ConcRun(
    function=my_async_function,
    thread=10,
    *args,
    **kwargs
)
```

**特点：**
- 使用信号量控制并发数量
- 所有任务同时启动并等待完成
- 返回所有任务的结果列表

### 2. SubTaskConcRun - 子任务并发执行
处理单个类实例中所有列表属性的并发任务。

```python
results = await RunMethod.SubTaskConcRun(
    func=my_async_function,
    items=class_instance,
    *args,
    **kwargs
)
```

**工作原理：**
1. 自动检测类实例中的列表/元组属性
2. 为列表中每个项目创建并发任务
3. 保持其他属性作为公共参数

### 3. MultiTaskConcRun - 多任务分批并发
处理大量任务的分批并发执行。

```python
async for batch_results in RunMethod.MultiTaskConcRun(
    func=my_async_function,
    items=task_list,
    thread=20,
    wait=0.5,
    *args,
    **kwargs
):
    # 处理每批结果
    for result in batch_results:
        print(f"处理结果: {result.response}")
```

**特点：**
- 分批处理，避免内存过载
- 支持批次间等待时间
- 返回生成器，逐批产出结果
- 自动处理线程数限制（1-200）

### 4. ReqConcRun - 请求并发执行
专为HTTP请求等场景设计的并发执行器。

```python
param = ReqConcParam(
    func=api_request_function,
    task=request_list,
    thread=15,
    wait=1.0
)

async for batch_results in RunMethod.ReqConcRun(param):
    for result in batch_results:
        response = result.result
        task_info = result.task
        # 处理响应
```

**特点：**
- 分批处理任务列表
- 支持批次间延迟
- 返回ReqConcResult对象，包含结果和任务信息

## 使用场景

### 1. API 批量请求
```python
# 批量API请求示例
async def make_api_request(url_data):
    # 实现API请求逻辑
    pass

urls = [{'url': 'https://api1.com'}, {'url': 'https://api2.com'}]
param = ReqConcParam(
    func=make_api_request,
    task=urls,
    thread=10,
    wait=0.5
)

async for batch in RunMethod.ReqConcRun(param):
    for result in batch:
        print(f"API响应: {result.result}")
```

### 2. 数据处理管道
```python
# 数据处理示例
class DataProcessor:
    def __init__(self, data_list, config):
        self.data_list = data_list  # 列表属性
        self.config = config

processors = [DataProcessor(data_chunk, config) for data_chunk in chunks]

async for batch in RunMethod.MultiTaskConcRun(
    func=process_data,
    items=processors,
    thread=5,
    wait=0.2
):
    # 处理每批数据处理结果
    pass
```

### 3. 文件批量处理
```python
# 文件处理示例
file_tasks = ['file1.txt', 'file2.txt', 'file3.txt']
param = ReqConcParam(
    func=process_file,
    task=file_tasks,
    thread=3,
    wait=0.1
)

async for batch in RunMethod.ReqConcRun(param):
    for result in batch:
        print(f"文件处理完成: {result.task}")
```

## 性能优化建议

### 1. 线程数设置
- 默认范围：1-200
- I/O密集型任务：建议20-50
- CPU密集型任务：建议与CPU核心数相当
- 根据目标服务器负载能力调整

### 2. 等待时间配置
- 避免对目标服务器造成过大压力
- 根据API限流策略设置合适的等待时间
- 建议0.1-2.0秒之间

### 3. 内存管理
- 使用生成器模式处理大量数据
- 及时处理批次结果，避免内存累积
- 考虑使用数据库或文件存储中间结果

## 错误处理

```python
import asyncio
import logging

async def safe_concurrent_run():
    try:
        param = ReqConcParam(
            func=my_function,
            task=task_list,
            thread=10,
            wait=0.5
        )
        
        async for batch in RunMethod.ReqConcRun(param):
            for result in batch:
                if hasattr(result.result, 'status') and result.result.status != 200:
                    logging.error(f"任务失败: {result.task}")
                    
    except Exception as e:
        logging.error(f"并发执行错误: {e}")
        # 实现重试逻辑或错误恢复
```

## 注意事项

1. **异步上下文**: 所有方法都需要在异步上下文中调用
2. **资源管理**: 确保正确关闭网络连接和文件句柄
3. **错误处理**: 实现适当的异常处理机制
4. **监控**: 建议添加日志记录和性能监控
5. **依赖**: 需要确保 `utils.http_client.AsyncResponse` 可用

## 版本兼容性

- 要求 Python 3.7+
- 依赖 asyncio 标准库
- 兼容主流异步HTTP客户端库

---

*此文档基于 concurrency_utils.py 代码分析编写，如有疑问请参考源代码实现*

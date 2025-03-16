# 集成TTS训练、微调、推理全流程到AI-Agent中

## Open-Manus

### Agent

- BaseAgent
- PlanningAgent
- ReActAgent
- SWEAgent
- ToolCallAgent

各个Agent的Prompt都在`prompt`文件夹中。

可用工具在`NEXT_STEP_PROMPT`：PythonExecute, FileSaver, BrowserUseTool, WebSearch, Terminate。

- PlanningAgent强调了多次**避免过度思考**，计划目标完成后立即`finish`停止输出。
- SWEAgent强调了代码格式(避免输出多余的空格导致python、cmd报错，并且给了一个模板`NEXT_STEP_TEMPLATE`)、代码单元性(禁止一次发出多个命令/调用多个工具，即必须等到环境返回结果后再继续)、交互式命令禁止

#### BaseAgent

Prompt、Planning、记忆更新、卡顿处理、执行

#### PlanningAgent

创建和管理计划以解决任务。

#### ReActAgent

Reasoning and Acting.

还没实现。

#### SWEAgent

Software Engineering.

还没实现。

#### ToolCallAgent

- `async def think(self) -> bool`：思考过程记录，选择工具；是否需要继续思考
- `async def act(self) -> str`：依次执行工具，将结果添加到Memory中，并返回
- `async def execute_tool(self, command: ToolCall) -> str`：执行单个工具，供`act`调用
  - 验证命令格式
  - 检查工具是否存在
  - 解析 JSON 参数
  - 执行工具并格式化结果
  - 处理特殊工具

- 是`PlanningAgent`的**基类**

### Tool Collection

- `basr.py`：基础工具，所有工具类均继承自此类。
- `tool_collection.py`：工具集合，应该是`tool`文件夹下的顶层代码。

### Global

#### LLM

- `AsyncOpenAI` and `AsyncAzureOpenAI` for LLM API
- `REASONING_MODELS = ["o1", "o3-mini"]`：后续可以开发DeepSeek R1？应该也兼容Openai的API

#### Schema

### External Packages

#### pydantic

- `BaseModel` for data validation
- `Field`：额外的验证规则和元数据。

#### tiktoken

tiktoken 是 OpenAI 开源的一个快速 BPE (Byte Pair Encoding) 分词器，用Rust编写。它主要用于计算文本的 token 数量，这对于在使用 OpenAI API 时估算成本和控制输入长度非常重要。

#### typing

静态类型检查，提高代码可读性与规范性。

- `Dict`
- `List`
- `Optional`: `None` or `Type`
- `Union`：`Type1` or `Type2`：`value: Union[int, str] = 123`

#### tenacity

通用retry逻辑

#### Method

`@classmethod` 是 Python 中的一个内置装饰器，它用于将一个类的方法转换为类方法。类方法与普通实例方法的主要区别在于：

1. **第一个参数接收类而非实例**：类方法的第一个参数通常命名为 `cls`（约定俗成），它接收的是类本身，而不是类的实例。

2. **可通过类直接调用**：类方法可以通过类名直接调用，而不需要创建类的实例。当然，也可以通过实例调用。

3. **可以访问和修改类级别的属性**：类方法能够访问和修改类的状态，这对于实现工厂方法、替代构造函数等模式非常有用。

类方法的典型应用场景包括：

- 创建工厂方法，提供多种方式实例化对象
- 实现替代构造函数，提供不同的初始化方法
- 处理与类相关但不依赖于特定实例的功能

例如：

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_string):
        # 从字符串格式创建 Date 对象
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)
    
    @classmethod
    def today(cls):
        # 创建表示今天的 Date 对象
        import datetime
        d = datetime.datetime.now()
        return cls(d.year, d.month, d.day)
```

这样就可以通过 `Date.from_string('2023-10-21')` 或 `Date.today()` 来创建实例，而不只是通过 `Date(2023, 10, 21)`。

---

- 类函数命名约定：

单下划线（_name）：表示内部使用的约定
双下划线（__name）：触发名称改写（name mangling），更强的"私有"暗示
双下划线包围（__name__）：特殊方法或属性（魔法方法）
`@abstractmethod`：抽象方法，必须在子类中实现
`@property`：属性方法，可以像属性一样访问，但实际上是一个方法
`@staticmethod`：静态方法，不需要实例化就可以调用
`@classmethod`：类方法，第一个参数是类本身
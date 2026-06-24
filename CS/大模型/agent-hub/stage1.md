###  Function calling
工具的意义在于让模型能不止知道训练数据，还能与外部系统交互访问训练数据之外的数据
函数调用！=工具调用，是工具调用的子集，当开发者使用 JSON Schema 严格定义输入输出规则时，使用的是“函数 (Function)”；当使用纯文本或直接调用官方原生内置工具时，你使用的是其他类型的“工具 (Tool)”。
#### 工具调用的5个标准流程
1. 初次请求，会在输入中带一个可用的工具列表
2. 模型决策与返回，模型发现只凭自身知识答不出来，会暂停作答，转而返回一个工具调用请求
3. 程序发现大模型需要工具的输出结果，就在本地运行工具
4. 将本地运行的结果作为一个新消息追加到上下文中继续追问大模型
5. 模型整合后发现可以回答，就回答了
### 会用一个 LLM API 完成普通对话
#### 单次对话
代码
```python
response = client.chat.completions.create(
    model=model,
    messages=messages
)

answer = response.choices[0].message.content
```
#### 多轮对话
每次保存模型的输出，添加进messages 里
```python
assistant_reply = response.choices[0].message.content  
  
messages.append({  
"role": "assistant",  
"content": assistant_reply  
})
```
### 会让模型输出结构化 JSON
普通对话的问题是：模型输出的是自然语言，程序不好处理。因此让大模型输出JSON会利于程序使用
有三种让模型输出JSON格式的方式
#### 1.只靠prompt 要求
类似
```
messages = [
{
"role": "system",
"content": "你是一个信息抽取助手。"
"你必须只输出 JSON，不要输出 Markdown，不要输出代码块，不要输出解释。"
},
{
"role": "user",
"content": """请分析这句话：我现在想学习如何让大模型 API 输出结构化 JSON。
请输出如下 JSON 格式：
{
"intent": "用户意图",
"topic": "主题",
"difficulty": "beginner/intermediate/advanced",
"keywords": ["关键词1", "关键词2"],
"need_step_by_step": true
}"""
}
]
```
#### 2.使用json mode
作用是：让模型输出一个合法 JSON 对象。看起来像是字典，实际还是str类型，使用json.loads（）进行解析后，将str转成python 字典,若不抛出JSONDecodeError错误，说明“是合法 JSON”，但依旧不能说明字段齐全、字段类型正确。字段结构要交给 Pydantic 或 Structured Outputs 校验。
做法是加入response_format={"type":"json_object"}，同时要在提示词里说明要输出json
使用Pydantic 校验
```python
from pydantic import BaseModel
from typing import Literal

class ToolCall(BaseModel):
    action: Literal["calculator", "search", "read_file"]
    arguments: dict
    need_tool: bool
```
使用`result = ToolCall.model_validate(data)`或`result = ToolCall.model_validate_json(content)`进行校验,输入类型分别是字典和字符串。如果数据合法，就会返回一个ToolCall 实例对象，数据不合法抛出`ValidationError`异常
####  3. JSON Schema
response=client.chat.completions.parse里加入字段(response_format=ToolCall)
result=response.choice[0].message.parsed

要注意有些模型不支持这种形式就要使用json mode+Pydantic实现


### 会定义一个工具函数，例如 search、calculator、read_file。
工具函数本质上就是一个普通的 Python 函数，是工具函数给 Agent 使用的外部能力。
例如：
```python
def calculator(expression: str) -> dict:
    ...
```
```python
def search(query: str) -> dict:
    ...
```
```python
def read_file(path: str) -> dict:
    ...
```
这些工具函数的共同点是：返回值最好统一为字典形式，并且都包含 `"success"` 字段，用来表示工具是否执行成功。
成功时可以返回：
```python
{
    "success": True,
    "result": ...
}
```
失败时可以返回：
```python
{
    "success": False,
    "error": "错误信息"
}
```
这样主程序可以统一判断：
```python
if tool_result["success"]:
    ...
else:
    ...
```
工具函数通常不会由我手动直接调用，而是通过工具执行器动态调用。
例如模型输出：
```python
{
    "action": "calculator",
    "arguments": {
        "expression": "25 * 48"
    },
    "need_tool": True
}
```
程序会根据 `action` 从 `TOOLS` 注册表中找到对应函数：
```python
TOOLS = {
    "calculator": calculator,
    "search": search,
    "read_file": read_file
}
```
然后通过：
```python
tool_func(**arguments)
```
把 `arguments` 字典展开为关键字参数。
例如：
```python
arguments = {
    "expression": "25 * 48"
}
```
那么：
```python
calculator(**arguments)
```

等价于：

```python
calculator(expression="25 * 48")
```

因此，`arguments` 字典里的 key 必须和工具函数的形参名一致。

例如函数定义是：

```python
def calculator(expression: str) -> dict:
    ...
```
那么模型输出的参数必须是：
```python
{
    "expression": "25 * 48"
}
```
不能写成：
```python
{
    "expr": "25 * 48"
}
```

否则会出现参数名不匹配错误。

总结：

工具函数负责完成具体能力，例如计算、搜索、读文件；`TOOLS` 注册表负责把工具名称映射到真实函数；`run_tool` 执行器负责根据 `action` 找到函数，并用 `tool_func(**arguments)` 动态调用工具。

### 会解析模型的 tool call / function call，会执行工具，并把工具结果喂回模型。
指的注意的是，`tools` 里写的 `parameters` 那段就是 JSON Schema,不需要使用json格式校验，也不需要设置Pydantic 校验了
知识点：
1. 能定义 tools 列表  
2. 知道 tools 只是给模型看的工具说明，不是本地函数本身  
3. 能把 tools=tools 传给模型  
4. 能读取 response.choices[0].message  
5. 能判断 message.tool_calls 是否存在  
6. 能从 tool_call.function.name 取出工具名  
7. 能从 tool_call.function.arguments 取出参数字符串  
8. 能用 json.loads() 把参数字符串转成 dict  
9. 能用 run_tool(name, arguments) 执行本地工具  
10. 能把工具结果用 role="tool" 和 tool_call_id 返回给模型  
11. 能再次调用模型生成最终回答
链路：
```
用户输入  
↓  
把 tools 列表传给模型  
↓  
模型返回 message.tool_calls  
↓  
从 tool_call 中取出 function.name  
↓  
从 tool_call 中取出 function.arguments  
↓  
json.loads(arguments)  
↓  
run_tool(name, arguments)  
↓  
把工具结果返回给模型  
↓  
模型生成最终自然语言回答
```

#### 工具的格式
```python
{
        "type":"function",
        "function":{
            "name":"calculator",
            "description":"一个计算器工具，可以计算数学表达式的结果",
            "parameters":{
                "type":"object",
                "properties":{
                    "expression":{
                        "type":"string",
                        "description":"需要计算的数学表达式，例如：2+2*3"
                    }
                },
                "required":["expression"],
                "additionalProperties":False
                }
        }
    }
```

#### 传给模型并读取message.tool_calls并将tool_calls传给模型
```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
    tools=tools,
)
```
message = response.choices[0].message
if message.tool_calls:
	messages.append(message)

#### 解析tool_calls并传给工具执行
有一些属性，比如id,function.name,function.arguments
`toolresult=Tools[function.name](**json.loads(functuon.arguments))`

#### 将工具结果返回给模型
messages.append({"rule":"tool","tools_call_id":"tool_call.id,"content":"json.dumps((tool_result, ensure_ascii=False) )})

### 会给 agent loop 加最大步数、超时和错误处理。







### 杂
#### direnv
#### uv


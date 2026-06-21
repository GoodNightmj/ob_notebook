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
[ChatGPT - stage1](https://chatgpt.com/s/t_6a3534a8b2548191a0bb2f5461b7ebce)

### 会让模型输出结构化 JSON
json的key必须是字符串，字符串必须用双引号

第一步：会让模型只输出 JSON  
在提示词里进行限制，无论是对system还是user

第二步：会用 json.loads() 解析  
输出的字符串能否使用json.loads（）进行解析，若不抛出JSONDecodeError错误，只能说明“是合法 JSON”，不能说明字段齐全、字段类型正确。字段结构要交给 Pydantic 或 Structured Outputs 校验。

第三步：会用 Pydantic 校验字段  
使用BaseModel这个类，将里面要加的具体限制加到类里
在通过model_validate(字典)函数将字典转换为python对象
第四步：了解 JSON mode  
就是在在response 里加入response_format={"type":"json_object"}的限制

第五步：了解 Structured Outputs
这里就是将response_format=AnasysisResult，并改变response = client.chat.completions.parse(model=model, messages=messages,response_format=AnasysisResult)
直接输出正确结果


### 会定义一个工具函数，例如 search、calculator、read_file。


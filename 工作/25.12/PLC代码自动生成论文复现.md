### 第一步：获取资源与环境准备

1. **获取源代码：** 作者在参考文献中提供了 GitHub 仓库地址。`https://github.com/hkoziolek/LLM-CodeGen-RAG` 。你可以先去查看该仓库。
    
2. **准备核心工具库：** 论文的原型是基于 Python 和 **LangChain** 框架构建的 。你需要安装以下 Python 库：
    
    - `langchain`：用于编排 LLM 和检索流程 。
        
    - `openai`：用于调用 GPT-4 和 Embeddings API 。
        
    - `faiss-cpu`：用作本地向量数据库 。
        
    - `pdfplumber`：用于解析 PDF 文档 。
        
    - `streamlit`（可选）：作者用它做了一个简单的 Web 界面来输入 Prompt 和展示结果 。
        
3. **获取数据源（OSCAT 库）：** 你需要下载 **OSCAT BASIC** 库的文档。
    
    - 这是一个开源库，文档通常是一个约 496 页的 PDF 文件 。
        
    - 你需要访问 OSCAT 官网（ 提到是 `http://www.oscat.de/`）下载该 PDF。
        
4. **准备测试环境（OpenPLC）：** 下载并安装开源的 **OpenPLC Editor** 。这是用来编译和运行你生成的代码的工具。
    

---

### 第二步：数据处理（构建向量库）

这是复现中最关键的一步，对应图 2 的上半部分。
磁向量、ai分块

1. **加载 PDF：** 使用 LangChain 的 `PDFPlumberLoader` 加载 OSCAT 的 PDF 文档 。这将提取文本和元数据。
    
2. **自定义分块（Critical Step）：** **注意：** 不要使用通用的字符分割器（如 CharacterTextSplitter）。
    
    - 作者编写了一个**自定义的文档分块器（Custom Document Chunker）**，基于**正则表达式（Regular Expression）** 。
        
    - **策略：** 正则表达式需要匹配 OSCAT 文档中每个功能块的小节编号/标题 。
        
    - **目的：** 确保每个切分出来的“块”包含了**一个完整的功能块说明**（包括输入、输出和功能描述），而不是在句子中间被切断 。这是为了保证 LLM 读到的是完整的说明书。
        
3. **生成嵌入（Embeddings）：** 使用 OpenAI 的 `text-embedding-ada-002` 模型将切分好的文本块转换为向量 。
    
4. **存储向量：** 将向量存储在本地的 **FAISS** 数据库中 。作者生成的数据库文件大小约为 4.5 MB 。
    

---

### 第三步：构建 RAG 生成流程

这一步对应图 2 的下半部分。

1. **设置 LLM：** 使用 `gpt-4` 模型（作者使用的是 `gpt-4-32k` version 0613） 。
    
    - **关键设置：** 将 Temperature 设置为 **0**，以确保输出尽可能确定 。
        
2. **构建检索器 (Retriever)：** 配置 LangChain 的 `RetrievalQA` 链 。
    
    - 使用基于分数的相似度搜索（Score-based similarity search）或者最大边际相关性（MMR）来查找与用户 Prompt 最相关的文档块 。
        
    - 配置检索器在返回结果时，包含检索到的源文档（Source Documents），以便在界面上展示参考资料 。
        
3. **设计提示词模板 (Prompt Engineering)：** 你需要完全复刻论文 **Figure 4** 中的 Prompt 模板 。
    
    - **核心指令：**
        
        - "Write a self-contained IEC 61131-3 ST function block"（写一个独立的功能块）。
            
        - "Use the following pre-specified function blocks if useful"（使用以下预定义的功能块）。
            
        - "Do not write code for the inner body of the function block"（不要写功能块内部的代码，即只做调用）。
            
        - "Do not include comments... Provide no explanations"（不要注释和解释，只给代码）。
            
    - **格式约束：** 强制要求输出格式为 `FUNCTION_BLOCK ... VAR_INPUT ... END_FUNCTION_BLOCK`。
        

---

### 第四步：测试与验证

1. **输入测试用例：** 使用论文第 5 节提供的具体 Prompt 进行测试：
    
    - _Test 1:_ "Sample an incoming signal at one-second intervals..."（采样与平均） 。
        
    - _Test 2:_ "Generate a sine wave..."（正弦波与阶梯函数） 。
        
    - _Test 3:_ "Create a PID controller..."（PID 控制器） 。
        
2. **验证生成结果：**
    
    - 查看生成的代码是否正确“实例化”了 OSCAT 库中的块（例如 `SH_1`, `FT_AVG`, `CTRL_PID`） 。
        
    - **注意手动修正：** 论文诚实地指出，GPT-4 生成的代码可能包含小错误（如参数名 `OUT` 写成了 `OUT_MAX`），复现时如果遇到编译错误，需要根据 OSCAT 手册手动微调代码 。
        
3. **OpenPLC 仿真：**
    
    - 将生成的 ST 代码复制到 OpenPLC Editor 中。
        
    - 你还需要导入依赖的 OSCAT 源代码（因为 OpenPLC 本身不带 OSCAT，你需要找一个 OpenPLC 版本的 OSCAT 库导入进去） 。
        
    - 编译并运行调试器，观察波形是否符合预期 。



国产自主产业链互联操作系统
企业业务集成和协同端对端工业互联新模式，老模式是什么



多模式工业智能体
多层级时空建模
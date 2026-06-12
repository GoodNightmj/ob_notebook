
## 1. 汇报目标

本次汇报围绕 Spec2Control 项目的主流程展开，目标是从“项目做什么、输入输出是什么、代码如何协同”三个角度说明该项目。具体目标如下：

1. 了解 Spec2Control 项目的基本工作流程：它如何把文字形式的控制要求转换成功能块图。
2. 弄清楚项目的输入文件、中间生成文件和最终输出文件分别是什么。
3. 看懂几个主要代码文件分别负责什么，以及它们之间是怎么连接起来的。

项目可以概括为：
Spec2Control 先读取 control narrative，也就是工业控制文字说明；然后用两组固定 prompt chain 调用大模型生成结构化文本；最后把文本形式的 FBD 转换成 OpenPLC Editor 可以打开的 PLCOpen TC6 XML 项目，也就是 `plc.xml`。

## 2. 项目整体流程

Spec2Control 的整体流程可以理解成三段：

```text
文字控制说明
  -> LLM 理解控制对象和控制策略
  -> LLM 生成文本形式的 FBD
  -> Python 后端转换为 PLCOpen XML
  -> OpenPLC Editor 打开项目文件夹
```

具体主链路如下：

```text
data/control-narratives/<案例名>/<案例名>.md
  -> src/spec2control_cli.py 读取并拆分章节
  -> 把单个章节文本填入 data/prompt-sets/contextgen.txt
  -> chapter_N/1_categorization.txt
  -> 把“章节文本 + 1_categorization.txt”填入 data/prompt-sets/openplc-fbd.txt
  -> chapter_N/2_control_logic.txt
  -> src/spec2control_backend.py 的 /transfer-openplc 接口接收 2_control_logic.txt
  -> src/export_openplc.py 转成 PLCOpen TC6 XML
  -> <案例输出目录>/plc.xml
```

这条链路里每个文件或路径保存的内容如下：

## 3. 输入文件是什么

项目最核心的输入是 control narrative 文件：

```text
data/control-narratives/<name>/<name>.md
```

每个 `<name>.md` 是一个工业过程案例，里面按章节写控制要求。CLI 会按 Markdown 章节标题拆分内容，每一章单独生成一个 Section，例如 `Section2`、`Section3`。
除了 control narrative，还有几类重要输入：
```text
data/prompt-sets/contextgen.txt
data/prompt-sets/openplc-fbd.txt
data/BASIC_LIB/specification/
data/BASIC_LIB/src/
data/process_config.yaml
```
其中：
- `contextgen.txt` 是第一组 prompt chain，负责识别当前章节需要哪些传感器、执行器和控制策略。
- `openplc-fbd.txt` 是第二组 prompt chain，负责生成文本形式的功能块图。
- `data/BASIC_LIB/specification/` 存放功能块说明资料，方便阅读和理解。
- `data/BASIC_LIB/src/` 存放导出 XML 时要嵌入或引用的功能块库 XML。
- `data/process_config.yaml` 是批处理配置，定义要跑哪些 narrative、哪些章节、输出目录等。
## 4. 功能块资料从哪里来
大模型的任务不是去网上查这些块，而是在 prompt 已经给定的可用块中选择和组合。后端调用 LLM API 时，也没有网页搜索、浏览器检索或向量数据库检索逻辑。
`data/BASIC_LIB/src/` 的作用主要出现在后端导出阶段：当 Python 生成 `plc.xml` 时，会加载这里的基础功能块库，把需要的功能块包含进 PLCOpen 项目。

## 5. 中间生成文件是什么

每跑一个章节，通常会生成一个章节目录：

```text
chapter_N/
  0_cn_chapter.txt
  1_categorization.txt
  2_control_logic.txt
  prompt_chain_log.md
  evaluation_results.txt
```

| 文件或路径                                    | 类型           | 里面存放的内容                                                                               |
| ---------------------------------------- | ------------ | ------------------------------------------------------------------------------------- |
| `data/control-narratives/<案例名>/<案例名>.md` | 输入数据文件       | 原始 control narrative，即文字形式的工业控制要求。通常按 Markdown 章节组织，每章描述一个控制单元或一段控制逻辑                 |
| `src/spec2control_cli.py`                | Python 代码文件  | 存的是读取配置、发现 narrative、解析章节、调用 prompt chain、保存中间文件、调用后端导出接口等流程控制代码。                     |
| `data/prompt-sets/contextgen.txt`        | Prompt 工作流文件 | 第一组 prompt chain，JSON 格式。里面存了多个 prompt 步骤，以及可用的传感器功能块、执行器功能块、控制策略说明和对应引脚表。            |
| `chapter_N/1_categorization.txt`         | 中间生成文件       | 第一组 prompt chain 的输出结果。里面通常是当前章节需要的功能块规格、引脚说明，告诉模型这一章需要哪些知识                           |
| `data/prompt-sets/openplc-fbd.txt`       | Prompt 工作流文件 | 里面存了生成文本 FBD 的多步 prompt，包括识别 Function Blocks、校验 tag、生成连接、生成报警和联锁逻辑、生成参数连接、输出变量、规则检查等。 |
| `chapter_N/2_control_logic.txt`          | 中间生成文件       | 第二组 prompt chain 的最终输出。描述最终 FBD 结构                                                    |
| `src/spec2control_backend.py`            | Python 代码文件  | 把已经拼好的 prompt 发给后                                                                     |
| `src/export_openplc.py`                  | Python 代码文件  | PLCOpen XML 导出器代码。                                                                    |
| `<案例输出目录>/plc.xml`                       | 最终输出文件       | 完整的 PLCOpen TC6 XML 项目。里面包含项目结构、POU、每个章节对应的 `SectionN`、功能块实例、变量、连线、参数连接以及必要的库信息。      |


## 6. 最终输出文件是什么

最终给 OpenPLC Editor 打开的核心文件是：

```text
<输出案例目录>/plc.xml
```

![80515524306e188b0a565ae04e12971a.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20260611134621939.png)

## 8. 两组 prompt chain 分别做什么

### 8.1 第一组：contextgen.txt

第一组 prompt chain 的目标不是直接生成 FBD，而是先回答：

> 当前章节后续生成 FBD 时，需要知道哪些功能块、哪些引脚、哪些控制策略？

它主要包含三个步骤：

| 步骤 | 作用 |
| --- | --- |
| `contextgen1-sensors` | 判断需要哪些传感器功能块，例如 `ANALOG_IN`、`DIGITAL_IN`、`BOOL_IN` |
| `contextgen1-actuators` | 判断需要哪些执行器功能块，例如电动阀、开关阀、电机、变频电机 |
| `contextgen1-strategies` | 判断当前章节属于什么控制策略，例如 PID、Cascade、Ratio、Duty-Standby、Split Range |

输出文件是：

```text
chapter_N/1_categorization.txt
```

这一步的意义是给第二阶段提供“上下文材料”。也就是说，第二阶段不只看到原始 control narrative，还会看到第一阶段挑出来的功能块规格和控制策略说明。

### 8.2 第二组：openplc-fbd.txt

第二组 prompt chain 才负责生成文本化 FBD。

它会在多个步骤里逐渐完善结果：

1. 识别需要实例化哪些 Function Blocks。
2. 检查是否遗漏 taglist 中的传感器、执行器、报警开关、联锁信号。
3. 创建主控制策略相关的块间连接。
4. 创建报警和联锁相关的逻辑函数与连接。
5. 识别参数值，例如设定值、报警上下限、比例系数等。
6. 创建输出变量。
7. 做规则检查和格式修正。

最终输出文件是：

```text
chapter_N/2_control_logic.txt
```

这份文件就是 Python 后端转换 XML 的直接输入。

## 9. 主要代码文件职责

### 9.1 src/spec2control_cli.py

这是 CLI 主入口，负责把整个流程串起来。

主要职责：

- 读取 `data/process_config.yaml` 或命令行参数。
- 找到 `data/control-narratives/` 下的 narrative 文件。
- 把 Markdown 文件拆分成章节。
- 对每个章节运行第一组 prompt chain。
- 保存 `1_categorization.txt`。
- 把原章节内容和第一阶段结果拼接起来，运行第二组 prompt chain。
- 保存 `2_control_logic.txt`。
- 调用后端 `/transfer-openplc` 接口，把文本 FBD 转为 PLCOpen XML。

可以把它理解成“流程调度器”。

### 9.2 src/spec2control_backend.py

这是 FastAPI 后端。

它主要提供两个关键能力：

1. `/qa`：接收 prompt，调用配置好的 LLM API，返回模型输出。
2. `/transfer-openplc`：接收文本化 FBD，调用 `export_openplc.py` 导出 OpenPLC 项目。

这里的 LLM 可以配置为 Azure OpenAI 或阿里云通义兼容接口。后端本身没有联网搜索功能，它只是把 prompt 发给模型。

### 9.3 src/export_openplc.py

这是核心转换器，负责把 `2_control_logic.txt` 转成 PLCOpen TC6 XML。

主要职责：

- 解析文本化 FBD 中的几个 section。
- 创建 PLCOpen 项目结构。
- 创建变量、功能块实例、IEC 61131-3 函数。
- 创建数据连接和参数连接。
- 加载 `data/BASIC_LIB/src/` 中的功能块库。
- 如果已有 `plc.xml`，就把新章节更新进去。
- 如果没有 `plc.xml`，就新建一个项目。
- 最终保存为 `<案例输出目录>/plc.xml`。

可以把它理解成“文本 FBD 到 XML 的编译器”。

### 9.4 src/autolayout_openplc_fbd.py

这个文件负责自动布局。

LLM 生成的是功能块和连接关系，但图在 OpenPLC Editor 里还需要坐标位置。`autolayout_openplc_fbd.py` 会根据块类型和连接关系安排位置，尽量让图从左到右、从输入到输出更容易阅读。

### 9.5 src/tc6_xml_v201.py

这是 PLCOpen TC6 v2.01 schema 生成的数据类文件，体量很大。

一般不需要手动改它。它的作用是让 Python 能用类型化对象创建符合 PLCOpen TC6 结构的 XML。


## 13. 汇报总结

Spec2Control 的核心思想是把自然语言控制说明拆成两个 LLM 阶段和一个确定性导出阶段。第一阶段让模型从固定的功能块和控制策略知识中挑出当前章节需要的上下文；第二阶段让模型基于章节原文和这些上下文生成文本化 FBD；最后由 Python 后端把文本 FBD 解析成 PLCOpen TC6 XML。

项目的输入是 `data/control-narratives/` 中的 control narrative，主要中间文件是每章的 `1_categorization.txt` 和 `2_control_logic.txt`，最终输出是案例目录根部的 `plc.xml`。OpenPLC Editor 打开时应选择包含 `plc.xml` 的案例文件夹。对于 `data/fbd-GPT5-generated/` 这批结果，每个案例只有一个合并后的 `plc.xml`，章节目录只保存中间结果，因此章节目录下没有 `plc.xml` 是正常现象。

## 后续打算
### 1.agent化
```
Narrative Reader Agent
  负责读章节、提取 tag、整理输入

Context Agent
  负责判断需要哪些功能块和控制策略

FBD Generation Agent
  负责生成 2_control_logic.txt

Validation Agent
  负责检查端口、类型、连接、XML 是否合法

Repair Agent
  根据错误信息让模型修正 FBD

XML Export Agent
  负责调用 export_openplc.py 生成 plc.xml
```
### 2.加入RAG技术
这个项目是在用固定知识库 + Prompt 注入。也就是把功能块类型、引脚、控制策略等知识提前写在 prompt 里，
模型从固定 prompt 里自己找知识，容易漏块、错端口、上下文臃肿。
加 RAG后：系统先按当前章节查出相关功能块、端口、策略和历史样例，再让模型生成，结果更准、更可扩展、更适合做论文实验。



展示智能化
展示准确性
运行思路
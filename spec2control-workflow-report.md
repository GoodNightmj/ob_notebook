# Spec2Control 项目工作流程汇报

日期：2026-06-11

## 1. 汇报目标

这份文档用于说明 Spec2Control 项目的基本工作流程，重点回答三个问题：

1. Spec2Control 如何把文字形式的控制要求转换成功能块图。
2. 项目的输入文件、中间生成文件和最终输出文件分别是什么。
3. 主要代码文件各自负责什么，以及它们之间如何连接。

结合前面讨论，可以先给出一句话总结：

> Spec2Control 先读取 control narrative，也就是工业控制文字说明；然后用两组固定 prompt chain 调用大模型生成结构化文本；最后把文本形式的 FBD 转换成 OpenPLC Editor 可以打开的 PLCOpen TC6 XML 项目，也就是 `plc.xml`。

## 2. 项目整体流程

Spec2Control 的整体流程可以理解成三段：

```text
文字控制说明
  -> LLM 理解控制对象和控制策略
  -> LLM 生成文本形式的 FBD
  -> Python 后端转换为 PLCOpen XML
  -> OpenPLC Editor 打开项目文件夹
```

更具体地说，可以先看一条主链路：

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

为了把每一步的输入、输出和提示词关系讲清楚，可以拆成下面这张表：

| 阶段 | 输入 | 使用的代码或 prompt | 输出 | 输出用途 |
| --- | --- | --- | --- | --- |
| 章节读取 | `data/control-narratives/<案例名>/<案例名>.md` | `src/spec2control_cli.py` | 每章的纯文本内容 | 给后续 prompt chain 使用 |
| 上下文生成 | 当前章节文本 | `data/prompt-sets/contextgen.txt` | `chapter_N/1_categorization.txt` | 告诉第二阶段本章需要哪些功能块、引脚和控制策略 |
| FBD 文本生成 | 当前章节文本 + `1_categorization.txt` | `data/prompt-sets/openplc-fbd.txt` | `chapter_N/2_control_logic.txt` | 作为后端生成 PLCOpen XML 的直接输入 |
| OpenPLC 导出 | `2_control_logic.txt` 的内容 | `/transfer-openplc` + `src/export_openplc.py` | `<案例输出目录>/plc.xml` | 给 OpenPLC Editor 打开的最终项目文件 |

这里要特别注意两个“输入拼接”：

第一，`contextgen.txt` 的输入主要是当前章节原文。CLI 会把章节内容替换到 prompt 里的占位符，例如：

```text
{{control-narrative}}
```

所以第一阶段实际发给大模型的是：

```text
contextgen 提示词模板 + 当前章节 control narrative
```

这一阶段输出的是上下文，不是 FBD。它回答的是：“这一章后面画图时应该知道哪些传感器块、执行器块、控制策略和引脚？”

第二，`openplc-fbd.txt` 的输入不是只有章节原文，而是：

```text
当前章节 control narrative + 第一阶段生成的 1_categorization.txt
```

CLI 会把这两部分合并后替换到第二组 prompt 里的占位符：

```text
{{specification}}
```

所以第二阶段实际发给大模型的是：

```text
openplc-fbd 提示词模板 + 当前章节原文 + 第一阶段功能块/策略上下文
```

这一阶段输出的 `2_control_logic.txt` 才是文本形式的功能块图。它通常包含这些固定小节：

```text
* Function Blocks *
* Variables *
* Functions *
* Data Connections *
* Parameter Data Connections *
```

最后，`2_control_logic.txt` 会被发送给后端 `/transfer-openplc` 接口。后端不是再让大模型生成 XML，而是调用 `export_openplc.py` 做确定性的解析和 XML 生成。也就是说，项目不是直接让大模型输出 `plc.xml`，而是让大模型输出一种更容易解析的中间文本，再由 Python 代码转换成标准 PLCOpen TC6 XML。

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

上午讨论过一个关键问题：功能块引脚等资料是不是大模型上网搜索来的？

结论是：不是。

当前项目更接近“固定知识库 + prompt 注入”的方式。也就是说，功能块类型、引脚名、数据类型和说明，大多已经写在 prompt 文件里。例如 `contextgen.txt` 里面直接列出：

```text
ANALOG_IN
BOOL_IN
DIGITAL_IN
MOTOR_ON_OFF
MOTOR_VSD
VALVE_ELECTRIC
VALVE_ON_OFF
PID_BASIC
RATIO_CONTROL
SPLIT_RANGE
VOTING_ANALOG
DUTY_STANDBY
```

并且每个块下面写有类似这样的引脚表：

```text
Direction,Pin Name,Type,Description
Input,PV_High,REAL,High alarm limit for PV.
Output,High_Alarm,BOOL,TRUE if PV exceeds PV_High limit.
```

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

各文件含义如下：

| 文件 | 含义 |
| --- | --- |
| `0_cn_chapter.txt` | 当前章节的原始控制说明文本 |
| `1_categorization.txt` | 第一阶段 context generation 的结果 |
| `2_control_logic.txt` | 第二阶段生成的文本化 FBD，是后续转 XML 的核心输入 |
| `prompt_chain_log.md` | 每一步 prompt 和模型输出日志，便于调试 |
| `evaluation_results.txt` | 对生成结果的评估结果，如果该批运行包含评估 |

其中最关键的是 `2_control_logic.txt`。它不是 XML，而是一种结构化文本，通常长这样：

```text
* Function Blocks *
ANALOG_IN FT-101
PID_BASIC FIC-104
VALVE_ELECTRIC FV-104

* Variables *
REAL FT101_FeedFlow_PV

* Functions *
OR ValveInhibit_OR

* Data Connections *
FT-101.PV, FIC-104.PV
FIC-104.XOUT, FV-104.Control_Signal

* Parameter Data Connections *
0.5, FIC-104.SP
TRUE, FT-101.Alarm_Enable
```

后端会解析这些 section，然后创建对应的 PLCOpen XML 元素。

## 6. 最终输出文件是什么

最终给 OpenPLC Editor 打开的核心文件是：

```text
<输出案例目录>/plc.xml
```

注意，OpenPLC Editor 通常不是直接选择 `plc.xml` 文件，而是在 `File -> Open Project` 里选择包含 `plc.xml` 的整个项目文件夹。

例如：

```text
data/fbd-GPT5-generated/ammonium-nitrates/
  plc.xml
  chapter_2/
  chapter_3/
  chapter_4/
  chapter_5/
  chapter_6/
```

打开时应选择：

```text
data/fbd-GPT5-generated/ammonium-nitrates/
```

而不是选择：

```text
data/fbd-GPT5-generated/ammonium-nitrates/plc.xml
```

也不是选择：

```text
data/fbd-GPT5-generated/
```

## 7. 为什么每个章节目录下没有 plc.xml

这是上午讨论的另一个重点。

在 `data/fbd-GPT5-generated/` 这批结果里，每个案例目录是一个完整 OpenPLC 项目。多个章节会合并到同一个案例根目录的 `plc.xml` 中，而不是每个章节单独生成一个 `plc.xml`。

例如：

```text
data/fbd-GPT5-generated/ammonium-nitrates/plc.xml
```

这个 `plc.xml` 里面包含多个章节对应的 POU，例如：

```text
Section2
Section3
Section4
Section5
Section6
```

所以章节目录：

```text
chapter_2/
chapter_3/
chapter_4/
```

主要保存该章节的 prompt 输入、LLM 输出和评估结果。它们不是独立的 OpenPLC 项目目录，因此里面没有 `plc.xml` 是正常的。

如果只跑一个章节，也可以生成可打开的项目。区别只是根目录 `plc.xml` 里只有一个 Section，例如 `Section2`。

## 8. 两组 prompt chain 分别做什么

### 8.1 第一组：contextgen.txt

第一组 prompt chain 的目标不是直接画图，而是先回答：

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

## 10. 代码之间如何连接

可以用下面这张文字图理解：

```text
spec2control_cli.py
  |
  | 读取 narrative，拆章节
  v
contextgen.txt prompt chain
  |
  | 生成 1_categorization.txt
  v
openplc-fbd.txt prompt chain
  |
  | 生成 2_control_logic.txt
  v
spec2control_backend.py
  |
  | /transfer-openplc
  v
export_openplc.py
  |
  | 解析文本 FBD，加载 BASIC_LIB，调用 autolayout
  v
plc.xml
  |
  v
OpenPLC Editor 打开案例文件夹
```

其中最重要的一条连接是：

```text
章节原文 + 1_categorization.txt -> openplc-fbd.txt -> 2_control_logic.txt
```

也就是说，第一阶段不是孤立结果，而是第二阶段的上下文输入。

## 11. fbd-GPT5-generated 文件夹如何理解

`data/fbd-GPT5-generated/` 是一批已经生成好的 GPT5 输出样例。它的结构是：

```text
data/fbd-GPT5-generated/
  ammonium-nitrates/
    plc.xml
    chapter_2/
    chapter_3/
    ...
  benzene-distillation/
    plc.xml
    chapter_2/
    chapter_3/
    ...
  coking-refinery/
    plc.xml
    chapter_2/
    chapter_3/
    ...
```

这里每个案例目录下面都有自己的 `plc.xml`。因此 OpenPLC Editor 应该打开具体案例目录，例如：

```text
data/fbd-GPT5-generated/ammonium-nitrates/
```

而不是打开整个 `data/fbd-GPT5-generated/`。

每个 `chapter_N/` 下没有 `plc.xml`，原因是这批结果采用“案例级合并项目”的结构。章节内容已经作为 `SectionN` 合并进案例根目录的 `plc.xml` 里。

## 12. 最小验收理解

如果要判断自己是否已经理解项目主流程，可以检查是否能回答下面几个问题：

1. 输入 control narrative 在哪里？
   - 在 `data/control-narratives/<name>/<name>.md`。

2. 哪个文件负责拆分章节？
   - `src/spec2control_cli.py`。

3. 两个 prompt chain 分别做什么？
   - `contextgen.txt` 负责选功能块和控制策略上下文。
   - `openplc-fbd.txt` 负责生成文本形式 FBD。

4. `2_control_logic.txt` 是什么？
   - 它是 LLM 生成的结构化文本 FBD，也是转 XML 的直接输入。

5. `/transfer-openplc` 接口做什么？
   - 它接收文本 FBD，调用 `export_openplc.py` 生成或更新 `plc.xml`。

6. `export_openplc.py` 如何生成 `plc.xml`？
   - 它解析文本 FBD，创建 PLCOpen 项目对象，加载 BASIC_LIB，并保存为 XML。

7. OpenPLC Editor 应该打开哪个路径？
   - 应该打开包含 `plc.xml` 的案例项目文件夹，而不是单独打开 `plc.xml` 文件，也不是打开章节目录。

## 13. 汇报时可以这样总结

Spec2Control 的核心思想是把自然语言控制说明拆成两个 LLM 阶段和一个确定性导出阶段。第一阶段让模型从固定的功能块和控制策略知识中挑出当前章节需要的上下文；第二阶段让模型基于章节原文和这些上下文生成文本化 FBD；最后由 Python 后端把文本 FBD 解析成 PLCOpen TC6 XML。

项目的输入是 `data/control-narratives/` 中的 control narrative，主要中间文件是每章的 `1_categorization.txt` 和 `2_control_logic.txt`，最终输出是案例目录根部的 `plc.xml`。OpenPLC Editor 打开时应选择包含 `plc.xml` 的案例文件夹。对于 `data/fbd-GPT5-generated/` 这批结果，每个案例只有一个合并后的 `plc.xml`，章节目录只保存中间结果，因此章节目录下没有 `plc.xml` 是正常现象。

## 论文流程图
**整体输入**
左边有三个来源：`IO-List`、`Control Narrative`、`P&IDs`。  
其中最核心的是 Control Narrative 控制叙述，也就是用自然语言写的控制需求。IO 清单和 P&ID 图可以提供辅助信息，比如有哪些仪表、阀门、泵、控制回路等。

**1. Planning 规划**
图中第 1 步先分析输入材料，决定采用什么处理策略。  
比如一篇控制叙述可能很长，不能直接整篇丢给 LLM，所以要判断：是整段处理、按章节切分，还是抽取某些 tag 相关内容。图里下面的 `Strategies` 就是在说这些策略：原样抽取、LLM 摘要后再处理、按事实抽取等。

**2. Control Narrative Extraction 控制叙述抽取**
第 2 步根据规划结果，从控制叙述里抽出当前要处理的部分，形成图中的 `Control Narrative Chunk`。  
也就是说，这一步把大文档切成适合生成一个 FBD 的小片段。图里的 `Process Loop` 表示如果有多个章节或多个 chunk，就会循环处理。

**3. LLM Context Generation 上下文生成**
第 3 步开始用 LLM 做工程理解，但它还不是直接生成控制图。  
它会结合 `Library Repository` 函数块库，检索可用函数块和控制策略，输出三类关键信息：
- `Required Library Function Blocks`：当前需求需要哪些函数块，比如传感器、阀门、PID、比值控制块等。
- `Identified Control Strategy`：识别控制策略，比如 PID、cascade、feedforward、ratio control、duty/standby 等。
- `Selected existing logic to attach to`：如果系统里已经有部分控制逻辑，就找出需要连接或复用的已有逻辑。

图中 `Control Engineer` 虚线监督这一步，表示工程师可以审查 LLM 识别得对不对。

**4. LLM Control Logic Generation 控制逻辑生成**
右侧大框就是第 4 步的内部细节。第 3 步生成的上下文会被 `Load into Context Window` 放进 LLM 上下文，然后 LLM 生成控制逻辑伪代码。
它内部依次做这些事：
1. `Instantiate Function Blocks`：实例化功能块，比如为 LT-104 创建一个 ANALOG_IN，为 FV-101 创建一个 VALVE_ELECTRIC。
2. `Create Connections`：建立功能块之间的连接，比如传感器 PV 输出连到控制器输入，控制器输出连到阀门命令。
3. `Parametrize Function Blocks`：填参数，比如报警上限、报警下限、设定值、使能标志。
4. `Create Interlocking Logic`：生成联锁逻辑，比如高液位报警时停泵、禁止开阀。
5. `Create Alarm Logic`：生成报警逻辑，比如超过温度上限触发 High Alarm。
6. `Validate Function Block Diagram`：检查生成的 FBD 是否完整、连接是否合理、函数块和参数是否符合库定义。

这一步的输出是图下方的 `Control Logic Pseudo Code`，还不是最终 XML。

**5. Pseudo Code Conversion 伪代码转换**
第 5 步是确定性程序处理，，LLM 到这里已经完成任务，后面用普通程序把 `Control Logic Pseudo Code` 转换成 `Function Block Diagram (XML)`。项目里，这一步对应把文本格式的 FBD 解析并导出为 PLCopen/OpenPLC XML，同时做自动布局。

**6. PLC Integrated Development Environment PLC 集成开发环境**
第 6 步把生成的 XML 导入 PLC IDE，比如 OpenPLC Editor。  
工程师可以在 IDE 里看到真正的图形化 FBD，检查连接、修改布局、补充逻辑、做测试。

**7. Runtime 运行时部署**
最后第 7 步，把已经检查和测试过的控制逻辑部署到 `Automation Controller`。  
控制器连接现场传感器和执行器，周期性执行这套逻辑，真正参与工厂控制。


## Spec2Control 复现

### 一、项目介绍
[项目链接](https://github.com/hkoziolek/Spec2Control)


我这段时间主要在复现 Spec2Control 这个项目。
这个项目的核心目标比较明确，就是将工业控制领域中的自然语言说明自动转换为 PLC 控制图。  
换句话说，在工业控制场景中，工程师通常会先用文字描述控制需求，例如温度过高需要报警、某些条件下执行器需要停止、系统需要满足特定联锁逻辑等。这些最初都是自然语言形式的需求描述，但在实际落地时，最终需要转化为结构化的控制逻辑，也就是 PLC 中常见的功能块图。

Spec2Control 做的事情，就是把前面的“文字需求”自动转换为后面的“控制图”。

从流程上看，这个项目大致分为三步：

第一步，是读取并理解文字说明。  
项目首先读取工业过程的文字描述，也就是 control narrative，然后从中识别控制系统涉及的关键元素和关系，例如传感器、执行器、控制器、报警条件、联锁逻辑以及控制策略等。本质上，这一步是在把一段自然语言拆解为“控制系统中有哪些元件，它们之间是什么关系”。

第二步，是生成结构化控制逻辑。  
在识别出这些元件和关系之后，项目会进一步生成控制逻辑。例如，需要不要使用 PID 控制，哪些信号应该连接到哪个功能块，哪些位置需要补充 OR、AND、NOT 等逻辑块，哪些参数需要设置上下限，以及哪些输出变量需要显式暴露出来。也就是说，这一步是在把“文字理解结果”进一步转换为“结构化控制逻辑”。

第三步，是导出为 OpenPLC 可打开的工程文件。  
项目最终会将生成结果导出为 PLCOpen XML，并进一步形成每个章节对应的 `SectionN.xml` 文件，目标是能够在 OpenPLC 中作为工程打开和查看。

---

### 二、我的工作

在复现过程中，我主要完成了以下几方面工作。

首先，我完成了项目源码的本地部署与依赖安装，并梳理清楚了项目的整体执行链路。目前项目已经能够正常启动后端服务，并能够对指定的 control narrative 章节执行完整的生成流程。

其次，在模型接口方面，原项目默认依赖 Azure OpenAI。由于我本地复现时并没有使用 Azure 环境，因此我对项目进行了适配改造，将其大模型接口替换为阿里云通义千问。目前，项目已经可以通过通义千问完成上下文生成和控制逻辑生成。

另外，在运行与导出阶段，我解决了项目输出目录与仓库文档描述不一致的问题。原始代码生成出的文件结构与官方 README 中给出的工程结构并不一致，缺少每个章节单独导出的 `SectionN.xml` 文件。针对这个问题，我补充实现了每章节独立 XML 的导出能力，并同步补充了 `beremiz.xml`，使得当前输出目录结构能够与官方 README 所描述的工程结构保持一致。

进一步测试后，我确认项目目前已经能够正常完成单章样例的处理流程，具体包括以下几个阶段：

- 上下文生成
- FBD 文本控制逻辑生成
- PLCOpen XML 导出
- 每章节 `SectionN.xml` 文件生成

---

### 三、目前发现的问题

不过，在进一步验证时，我发现项目虽然能够完成生成流程，但生成结果目前仍然无法在 OpenPLC 中正常打开。

一开始，我定位到一个比较明显的问题：部分生成结果中存在同名实例重复定义。从 XML 语法角度来看，这类文件本身仍然是合法的，但 OpenPLC 在实际加载时无法接受这种重复定义。这个问题目前已经被我定位并完成处理。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20260407132020107.png)

但是，在修复这一问题之后，工程仍然无法正常打开。于是我又进一步使用 OpenPLC 去尝试打开项目仓库中 `data/fbd-baseline/` 目录下的 baseline 文件。按项目说明，这部分内容属于人工审查过的 baseline FBD，理论上应该能够被 OpenPLC 正常打开；但实际测试结果是，这部分内容同样无法直接打开。

在此基础上，我又使用 OpenPLC 新建了一个空项目，并将其工程目录结构与该项目生成结果进行对比。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20260407132409306.png)对比后可以看到，两者之间存在比较明显的工程格式差异。再结合当前使用的 OpenPLC Editor 4.1.4 的实际表现——它不仅打不开我生成的工程目录，连仓库自带的 baseline 目录也无法通过 Open Project 直接打开——因此我目前判断，这个问题并不只是简单的“复现失败”，而更可能是旧版工程格式与新版编辑器之间存在兼容性问题。

另外，论文中提到他们使用的是 GPT-5 作为 LLM 模型。既然使用的是较新的模型，理论上不太可能搭配一个非常陈旧且不兼容的 OpenPLC 环境。因此，我目前怀疑这个项目本身可能还没有完全成熟，或者其工程导出链路还存在未解决的问题。基于这一判断，我已经向项目作者提交了[issue](https://github.com/hkoziolek/Spec2Control/issues/1) ，目前还没有收到回应。

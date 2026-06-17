# Spec2Control Demo Runbook

我今天演示的是 Spec2Control。这个项目的目标很直接：把文字形式的工业控制要求，也就是 control narrative，转换成 PLC 编程中可用的 IEC 61131-3 Function Block Diagram，并最终导出为 OpenPLC Editor 可以打开的 PLCOpen TC6 XML 项目。
也就是说，工程师原来写的是一段控制说明，比如泵什么时候启动、阀门什么时候打开、什么条件下报警或联锁。Spec2Control 希望把这类自然语言需求，自动转换成更接近 PLC 工程交付物的功能块图。


---

**画面：项目根目录或 README.md**
配音：
接下来我按照实际运行链路来讲整个流程。我会讲到哪一步，就切到对应的文件或输出目录，让大家看到自然语言控制叙述是怎样一步一步进入系统，最后变成 OpenPLC Editor 可以打开的功能块图项目。
这个项目的主线可以先概括成一句话：从 control narrative 开始，经过两阶段 prompt 生成文本化功能块图，再由后端转换器导出 PLCOpen XML。

---

**画面：打开 `data/control-narratives/demo-process/demo-process.md`**

配音：

第一步，我们先看系统的输入。

这里打开的是：

```
data/control-narratives/demo-process/demo-process.md
```

这就是 control narrative，也就是自然语言控制叙述。它不是 PLC 程序，也不是 XML，而是一份工程师能读懂的控制说明。

这里面会写清楚这个工艺段有哪些设备和仪表，比如液位、流量、压力、温度信号；有哪些执行器，比如泵、阀门、搅拌器；以及这些设备在什么条件下启动、停止、报警或联锁。

比如 `LT-201` 代表现场的液位变送器，`FIC-201` 代表流量控制回路，`PMA-201` 和 `PMB-201` 代表两台主备输送泵，`XV-201` 代表开关阀。

所以这一层解决的是“我要控制什么”的问题。输入还是工程语言，不是 PLC 图纸。

---

**画面：滚动到第 2 章**

配音：

这次 demo 我主要处理第 2 章。第 2 章包含比较完整的控制对象：模拟量输入、PID 控制、主备泵、开关阀、急停、报警和联锁。

我选择这一章，是因为它能比较完整地展示从自然语言到功能块图的转换过程。

---

**画面：打开 `src/spec2control_cli.py`，只展示文件整体，不讲函数**

配音：

接下来，系统需要读取这个输入文件，并决定要处理哪个章节。这里对应的是 CLI 文件：

```
src/spec2control_cli.py
```

这个文件可以理解成整个流程的调度入口。它负责找到 control narrative 文件，把 Markdown 按章节拆开，然后根据命令行参数选择要处理的章节。

在这个 demo 里，我会运行：

```
.\process.bat --narrative demo-process --chapters 2
```

这条命令的意思是：选择 `demo-process` 这个案例，只处理第 2 章。

CLI 做完这一步之后，会把第 2 章单独保存到本次运行的输出目录里。

---

**画面：打开输出目录中的 `chapter_2/0_cn_chapter.txt`**

配音：

这里可以看到第一个中间文件：

```
chapter_2/0_cn_chapter.txt
```

这个文件不是模型生成的，而是从刚才的 Markdown 输入里截取出来的第 2 章原文。

也就是说，从这一步开始，后续 prompt 真正处理的不是整个 `demo-process.md`，而是这个已经拆出来的章节文件。

所以到目前为止，流程是：

```
demo-process.md
  -> CLI 读取并拆章节
  -> chapter_2/0_cn_chapter.txt
```

---

**画面：打开 `data/prompt-sets/contextgen.txt`**

配音：

接下来进入第一阶段 prompt。对应的文件是：

```
data/prompt-sets/contextgen.txt
```

第一阶段的目标不是直接生成最终功能块图，而是先理解这一章的控制上下文。

它会从 `0_cn_chapter.txt` 里识别这一章有哪些传感器、执行器、控制回路、报警、联锁，以及可能需要哪些功能块。

这里要注意，项目不是让大模型上网查功能块知识，而是把支持的功能块类型、引脚说明和控制策略写在 prompt 里，作为固定上下文提供给模型。

比如模型需要判断：液位变送器通常可以映射到模拟量输入功能块，流量控制回路可能需要 PID 功能块，两台泵可能需要主备切换逻辑，急停和高高液位这类条件可能需要进入联锁逻辑。

---

**画面：打开 `chapter_2/1_categorization.txt`**

配音：

第一阶段运行完之后，会生成：

```
chapter_2/1_categorization.txt
```

这个文件可以理解成“生成 FBD 前的工程分析结果”。

它不是功能块图本身，也不能直接导出 OpenPLC XML。它更像是一份中间理解结果：把原始自然语言里分散的控制对象、控制策略和安全条件整理出来，方便下一步生成更结构化的功能块图。

到这里，流程变成：

```
0_cn_chapter.txt
  -> contextgen.txt
  -> 1_categorization.txt
```

---

**画面：打开 `data/prompt-sets/openplc-fbd.txt`**

配音：

接下来是第二阶段 prompt。对应的文件是：

```
data/prompt-sets/openplc-fbd.txt
```

这一阶段的任务，是把控制需求转换成文本形式的 Function Block Diagram。

它的输入有两部分：第一部分是原始章节，也就是 `0_cn_chapter.txt`；第二部分是第一阶段的分析结果，也就是 `1_categorization.txt`。

也就是说，第二阶段不是只看原始文字，而是带着前面整理好的工程上下文来生成 FBD。

---

**画面：打开 `chapter_2/2_control_logic.txt`**

配音：

第二阶段运行完之后，会生成：

```
chapter_2/2_control_logic.txt
```

这是整个流程里最关键的中间文件。

它已经不是普通自然语言，而是一种结构化的文本功能块图。里面通常会包括功能块实例、变量、基础逻辑函数、数据连接和参数连接。

比如这里会说明要创建哪些功能块实例，哪些现场 tag 对应哪些功能块；还会说明变量和功能块之间怎么连接，报警阈值、定时器参数、PID 设定值这类参数应该接到哪里。

可以把它理解成自然语言和 PLCOpen XML 之间的桥梁。

到这里，LLM 的核心工作基本完成了。后面系统不再让模型直接写最终 XML，而是把这个结构化文本交给后端转换器。

---

**画面：打开 `src/spec2control_backend.py`**

配音：

接下来进入后端。对应文件是：

```
src/spec2control_backend.py
```

这个后端承担两个角色。

第一，它负责接收 prompt 请求并调用 LLM，所以前面两阶段 prompt 都会通过后端完成模型调用。

第二，它负责接收已经生成好的 `2_control_logic.txt`，然后发起 OpenPLC 导出流程。

所以这里是一个分界点：前面是 LLM 负责理解和生成结构化文本，后面是程序负责把结构化文本转换成工程文件。

---

**画面：打开 `src/export_openplc.py`**

配音：

真正负责转换 PLCOpen XML 的文件是：

```
src/export_openplc.py
```

这个转换器会读取 `2_control_logic.txt`，解析里面的功能块、变量、函数、连线和参数，然后组装成 OpenPLC Editor 可以识别的 PLCOpen TC6 XML 项目。

这里的重点是：最终 XML 不是由模型自由生成，而是由 Python 转换器根据固定规则生成。

这样做的好处是，最终工程文件的结构更可控，也更方便排查问题。

---

**画面：打开 `data/BASIC_LIB/src/` 目录**

配音：

转换过程中还会用到功能块库。这里打开的是：

```
data/BASIC_LIB/src/
```

这个目录里存放的是功能块类型本身的 XML 定义，比如模拟量输入、PID 控制、开关阀、电机控制、主备切换等。

可以这样理解：这些 XML 文件是功能块模板，而 `2_control_logic.txt` 里写的是要创建哪些具体实例，以及这些实例之间怎么连接。

比如 `ANALOG_IN` 是模板，`LT-201` 是这个模板创建出来的一个现场实例。`PID_BASIC` 是模板，`FIC-201` 是具体的流量控制回路实例。

---

**画面：打开输出目录 `data/fbd-llm-output/demo-process_<timestamp>/`**

配音：

转换完成后，系统会在输出目录里生成本次运行结果。

这里可以看到几个关键文件：

```
chapter_2/0_cn_chapter.txt
chapter_2/1_categorization.txt
chapter_2/2_control_logic.txt
chapter_2/Section2.xml
plc.xml
```

它们刚好对应整个流程。

`0_cn_chapter.txt` 是原始章节。  
`1_categorization.txt` 是第一阶段理解结果。  
`2_control_logic.txt` 是第二阶段生成的文本化 FBD。  
`Section2.xml` 是单章节 PLCOpen XML。  
`plc.xml` 是最终 OpenPLC 项目文件。

---

**画面：打开 OpenPLC Editor，选择 Open Project**

配音：

最后一步，我们用 OpenPLC Editor 打开结果。

这里要注意，应该选择整个输出目录，而不是单独选择 `plc.xml` 文件。

也就是说，在 OpenPLC Editor 里选择 File -> Open Project，然后选择类似这样的目录：

```
data/fbd-llm-output/demo-process_<timestamp>/
```

---

**画面：OpenPLC Editor 中打开 `Section2`，展示 FBD**

配音：

打开之后，在项目树里进入 `Section2`，就可以看到生成出来的 Function Block Diagram。

现在屏幕上的功能块图，就是从最开始那份自然语言 control narrative，经过两阶段 prompt 和后端 XML 转换器生成出来的。

最后我们再把整条链路串一下：

```
demo-process.md
  -> CLI 拆出第 2 章
  -> 0_cn_chapter.txt
  -> 第一阶段 prompt 生成 1_categorization.txt
  -> 第二阶段 prompt 生成 2_control_logic.txt
  -> 后端转换
  -> plc.xml
  -> OpenPLC Editor 查看功能块图
```

这个设计的关键点是：LLM 负责理解控制叙述，并生成可读、可检查、可解析的文本化 FBD；最终 PLCOpen XML 由确定性的 Python 转换器生成。这样既利用了大模型的语言理解能力，也保留了工程交付物生成过程的可控性。

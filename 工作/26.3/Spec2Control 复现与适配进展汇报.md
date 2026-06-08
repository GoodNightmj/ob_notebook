[项目链接](https://github.com/hkoziolek/Spec2Control)
# Spec2Control 复现与适配进展汇报

## 1. 当前已完成工作

### 1.1 项目环境梳理与依赖安装

已完成 Spec2Control 项目的基础环境搭建，梳理了项目整体运行流程，包括：

- 后端服务启动方式
- CLI 处理链路
- 大模型调用位置
- OpenPLC 导出逻辑

同时完成项目依赖安装与运行环境准备，主要依赖包括：

- fastapi
- uvicorn
- openai
- pydantic
- python-dotenv
- PyMuPDF
- lxml
- xsdata
- PyYAML
- requests

### 1.2 大模型接口替换与适配

原项目默认使用 Azure OpenAI。为满足本地复现需求，已将项目改造为支持阿里云通义千问接口，主要工作包括：

- 梳理原项目中 Azure OpenAI 的调用位置
- 修改后端模型客户端初始化逻辑
- 调整 CLI 中的模型配置读取方式
- 补充阿里云通义所需环境变量
- 保留原有 Azure 配置兼容能力，避免对原项目结构造成破坏

目前已完成的通义配置如下：

```
LLM_PROVIDER=aliyun

ALIYUN_TONGYI_API_KEY=

ALIYUN_TONGYI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1

ALIYUN_TONGYI_MODEL_NAME=qwen-plus
```
### 1.3 处理流程联调

已完成项目基础联调，能够：

- 正常启动后端服务
- 正常识别控制叙述样例文件
- 正常进入大模型处理流程
- 输出初步生成结果到指定目录
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20260321184737313.png)
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20260321183142541.png)
说明项目主流程已经基本跑通，当前工作重点已从“能否运行”转向“输出结果是否完整、是否符合官方说明”。

## 2. 当前发现的关键问题

在实际测试中发现，项目生成结果与官方文档描述不完全一致。

### 2.1 官方文档中的目标输出结构

官方文档说明生成目录应类似于：

```
data/fbd-llm-output/
└── ammonium-nitrates_1003_143022/    # ← Open this folder in OpenPLC
    ├── plc.xml                        # Combined project file
    ├── chapter_2/
    │   ├── Section2.xml               # Individual chapter XML
    │   └── 2_control_logic.txt        # Textual FBD notation
    └── chapter_3/
        ├── Section3.xml
        └── 2_control_logic.txt
```

即：

- 顶层应有完整工程文件 plc.xml
- 每个章节目录下还应有单独的 SectionN.xml

### 2.2 当前实际生成结果

目前实际生成结果为：

data/fbd-llm-output/ammonium-nitrates_0321_175751/
├── plc.xml
└── chapter_2/
    ├── 0_cn_chapter.txt
    ├── 1_categorization.txt
    ├── 2_control_logic.txt
    └── prompt_chain_log.md

当前问题是：

- 顶层 plc.xml 已成功生成
- 中间处理文件已成功生成
- 但每个章节目录下缺少官方文档中提到的 SectionN.xml

这意味着当前输出结果还不能完全按文档描述直接用于 OpenPLC 工程级导入。

## 3. 已完成的问题定位工作

针对上述问题，已经完成以下排查工作：

- 检查 CLI 输出目录生成逻辑
- 检查后端 /transfer-openplc 接口处理逻辑
- 检查 export_openplc.py 中 XML 导出实现
- 对比官方文档与实际代码行为差异
- 确认问题不属于运行失败，而是导出逻辑未完整覆盖单章节 XML 输出

目前已经定位到：  
项目现有逻辑主要负责生成顶层合并工程 plc.xml，但没有完整实现每个章节独立 SectionN.xml 的导出流程。

## 4. 下一步计划

下一步准备继续推进以下工作：

- 深入阅读 export_openplc.py 的导出实现
- 补充每个章节单独 XML 文件的导出能力
- 保证最终输出同时包含：
    - 顶层完整工程 plc.xml
    - 各章节独立 SectionN.xml
- 完成修改后重新进行单章节和多章节回归测试
- 验证生成结果能否按官方说明直接导入 OpenPLC

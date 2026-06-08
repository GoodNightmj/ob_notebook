abstract、intorduction、colutions都看（挑着与自己相关的看）

## 📄 同学A任务文档：数据库构建与产品需求理解 王梓伊

### 📘 阅读内容：

- 2.1 Control architecture
- 2.2 PLC code formats and standards
- 2.3 Semantics for PLC modelling
- 3.1 Overall structure（概览）
- 3.1.1 Establishment of the database for PLC code
- 3.1.2 Understanding the product requirement
- 4.1 Establishment of the database for PLC and manufacturing information
- 4.2 Understanding the product requirements

### 🎯 任务目标：
- 理解整套PLC自动编排框架的总体流程（输入、处理、输出）
- 掌握如何构建支持自动化生成的数据库，包括：
  - 任务、产品、BoP、BoR、能力模型、PLC库的语义建模
  - 使用Ontology（CCO/ASO）和Graph数据库（Neo4j）
- 掌握BoP/BoR的自动推理流程：
  - 规则推理
  - 语义推理
  - 图嵌入推理（GraphSAGE）

### 📝 输出要求：
- 总结数据库设计的7类核心内容（任务、产品、BoP、BoR、能力、容量、PLC库）
- 梳理算法1的推理流程，并配合伪代码说明三个推理方法的适用场景
- 列出本体建模中常用的概念及其层次结构（如“Skill”与“Capability”的区别）

### 🖼 插图建议：
- 方法总览框架图（Fig.1）
- 数据库与知识图谱结构图（可参考 Fig.9）
- 推理流程图（基于Algorithm 1）

---

## 📄 同学B任务文档：仿真环境构建与PLC代码自动生成 吕盈忱

### 📘 阅读内容：
- 2.6 Graph Neural Network (GNN) and GraphSAGE model
- 2.7 Human–machine coexistence in manufacturing
- 2.8 Existing gaps
- 3.1.3 Building/updating the simulation process
- 3.1.4 Generation of the PLC control code
- 3.1.5 Conversion/check and update of the PLCopen XML
- 4.3 Building the simulation environment
- 4.4 Generation of the PLC code
- 4.5 Conversion/check of the PLCopen XML

### 🎯 任务目标：
- 理解如何基于BoP/BoR自动更新或构建仿真环境（Tecnomatix、JSON文件、.NET API）
- 掌握如何通过“技能模型”映射生成PLC代码：
  - 映射规则设计（参见Table 1）
  - 代码模块复用与组合
  - Siemens-compliant XML 生成流程
- 理解PLCopen XML的作用与生成过程

### 📝 输出要求：
- 总结仿真环境更新流程中涉及的输入信息（JSON结构）与自动/手动更新方式
- 梳理代码生成的两步：
  1. 技能模型 ↔ 映射规则 ↔ PLC库调用
  2. PLC代码 → Siemens XML → PLCopen XML
- 列举一个操作类型（如“机器人装配”）的PLC动作映射逻辑

### 🖼 插图建议：
- 仿真环境构建流程图（Fig.3）
- 代码生成流程图（Fig.2、Fig.14）
- PLCopen XML结构框图

---

## 📄 同学C任务文档：PLC代码测试与虚拟调试 毛剑

### 📘 阅读内容：
- 2.4 PLC code testing
- 2.5 Dynamic software reconfiguration
- 3.1.6 PLC code testing
- 3.1.7 Virtual commissioning and deployment
- 4.6 PLC code testing
- 4.7 Virtual commissioning

### 🎯 任务目标：
- 掌握PLC代码自动测试的流程与方法：
  - 覆盖测试（BC、ICC、CCC）
  - 变异测试
  - 测试用例生成 + Java测试器 + TIA Test Suite验证
- 理解如何利用虚拟调试（VC）完成代码的部署验证：
  - 软件/硬件闭环（SiL/HiL）
  - TIA Portal + PLCSIM Advanced

### 📝 输出要求：
- 总结FBD测试指标含义与差异
- 梳理测试流程各步骤：PLC代码 → Test Cases → 测试执行 → 输出断言
- 总结虚拟调试所用软件组件、流程与关键注意事项（如内存、干扰、接线检查）

### 🖼 插图建议：
- PLC测试流程图（Fig.4、Fig.15）
- 覆盖率测试用例图示（Fig.16）
- 虚拟调试与部署流程图



标题：
三到四个子标题
每个标题一个方案

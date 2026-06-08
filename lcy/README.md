# 地球化学数据分析与智能分类项目 (2025)

## 1. 项目概述

本项目旨在处理输入的地球化学数据Excel文件，执行探索性数据分析，进行特征工程，训练多种机器学习模型（随机森林、支持向量机、深度神经网络）以实现样品的智能分类（例如，区分“富铜”与“富金”样品），评估模型性能，使用SHAP等方法解释模型预测结果，并最终将预测流程打包成一个Streamlit图形用户界面（GUI）应用程序和一个可工作的Docker镜像。
项目采用模块化结构，每个主要的数据科学工作流程步骤都有专门的脚本对应。
***

## 2. 项目结构

```text
你的项目文件夹/
├── app.py                      # Streamlit GUI 应用程序脚本
├── Dockerfile                  # 用于构建 Docker 镜像的配置文件
├── requirements.txt            # Python 包依赖列表
├── .dockerignore               # Docker 构建时忽略的文件列表
├── data/                       # 存放原始输入数据
│   └── 2025-Project-Data.xlsx  # 示例输入 Excel 文件 (请根据实际情况重命名)
├── src/                        # 存放各个模块的 Python 源代码脚本
│   ├── data_processing.py        # 数据预处理与QC
│   ├── exploratory_analysis.py   # 探索性数据分析
│   ├── feature_engineering.py    # 特征工程
│   ├── model_training.py         # 模型训练
│   ├── model_evaluation.py       # 模型评估
│   └── model_interpretability.py # 模型可解释性分析
├── processed_data/             # data_processing.py 脚本的输出
│   └── processed_data.csv
├── features_engineered_data/   # feature_engineering.py 脚本的输出
│   └── features_engineered_data.csv
├── model_outputs/              # 保存训练好的模型、预处理器和相关图表
│   ├── tree_model_analysis/      # 树模型的分析结果
│   │   ├── random_forest_model.joblib
│   │   └── rf_logloss_vs_num_trees.png
│   ├── dnn_model_analysis/         # DNN 模型的分析结果
│   │   ├── best_dnn_model.keras
│   │   └── keras_tuner_logs/     # KerasTuner 的日志
│   ├── svm_rbf_model.joblib
│   ├── standard_scaler.joblib
│   ├── label_encoder.joblib
│   ├── evaluation_plots/         # model_evaluation.py 输出的评估图表
│   └── interpretability_plots/   # model_interpretability.py 输出的可解释性图表
├── plots/                      # 通用图表 (例如，来自 exploratory_analysis.py)
│   └── exploratory_analysis/
└── README.md                   # 本说明文件
```

***

## 3. 环境设置说明

### 先决条件
* Python (例如 3.9 版本 - 请确保与你的开发环境一致)
* Conda (Miniconda 或 Anaconda) 用于环境管理
* Docker Desktop (用于 Docker 镜像的构建和运行 - 此为可选的加分任务)
* C++ 编译器 (如 Microsoft Visual C++ Build Tools) - 如果 `scikit-bio` 等包需要从源码编译，则可能需要（通常 Conda 会处理好）。

### 使用 Conda 创建环境
强烈建议使用 Conda 环境来管理项目的依赖。

1.  **创建新的 Conda 环境:**
    打开 Anaconda Prompt 或你的终端，运行：
    ```bash
    conda create -n geochem_env python=3.9
    # 将 'geochem_env' 替换为你偏好的环境名称
    # 将 '3.9' 替换为你实际使用的 Python 版本
    ```
2.  **激活环境:**
    ```bash
    conda activate geochem_env
    ```
3.  **安装依赖:**
    项目根目录下的 `requirements.txt` 文件列出了所有必需的包。
    如果你还没有这个文件，可以从你当前正常工作的 Conda 环境中生成它：
    ```bash
    # 确保 'geochem_env' 已激活且所有包已安装
    pip freeze > requirements.txt
    ```
    然后，在新环境中通过以下命令安装依赖：
    ```bash
    pip install -r requirements.txt
    ```
    关键库包括 (请确保它们在 `requirements.txt` 中，或在你的 Conda 环境中手动安装它们):
    * `pandas`
    * `numpy`
    * `scipy`
    * `openpyxl` (用于读取 Excel 文件)
    * `scikit-learn`
    * `scikit-bio` (用于 CLR 转换)
    * `matplotlib`
    * `seaborn`
    * `tensorflow` (用于 Keras DNN 模型)
    * `keras-tuner`
    * `shap` (用于模型可解释性)
    * `streamlit` (用于 GUI 应用程序)
    * `joblib` (用于保存/加载 scikit-learn 模型)

    或者，你可以尝试用 Conda 直接安装一些核心包：
    ```bash
    # (geochem_env) C:\...>
    conda install pandas numpy scipy openpyxl scikit-learn scikit-bio matplotlib seaborn tensorflow joblib streamlit
    pip install keras-tuner shap
    ```
    *(请根据 `requirements.txt` 的内容或你偏好的安装方式进行调整)*

## 4. 数据说明
* 主要的输入数据期望是一个位于 `data/` 目录下的 Excel (`.xlsx`) 文件。
* 脚本中使用的示例文件名是 `2025-Project-Data.xlsx`。**如果你的文件名或位置不同，请相应更新脚本中的文件路径。**
* Excel 文件应包含元素成分数据和一个用于分类的 `Label` 列。脚本中 `ELEMENT_COLUMN_BASENAMES` 和 `TARGET_COLUMN` 变量定义了期望的列名。

## 5. 工作流程和脚本执行

请在激活 Conda 环境后，从项目的根目录按顺序运行以下脚本。

1.  **数据预处理与质量控制 (`src/data_processing.py`)**
    * **目的**: 从 Excel 加载原始数据，验证数据类型和单位，处理缺失值（为CLR转换填充0），执行CLR转换，标记异常值，并保存处理后的数据。
    * **运行命令**:
        ```bash
        python src/data_processing.py
        ```
    * **输入**: `data/2025-Project-Data.xlsx` (或你配置的路径)
    * **输出**: `processed_data/processed_data.csv` (包含原始数据、异常值标记和 `_clr` 后缀的列)

2.  **探索性数据分析 (`src/exploratory_analysis.py`)**
    * **目的**: 生成各种图表（成对散点图矩阵、相关性热图、PCA双标图、地球化学比率图）以理解数据特征。
    * **运行命令**:
        ```bash
        python src/exploratory_analysis.py
        ```
    * **输入**: `processed_data/processed_data.csv`
    * **输出**: 图表保存在 `plots/exploratory_analysis/` 目录下。

3.  **特征工程 (`src/feature_engineering.py`)**
    * **目的**: 执行单位协调（例如 wt% 转 ppm），创建新的地球化学比率特征，并对选定特征应用对数变换。
    * **运行命令**:
        ```bash
        python src/feature_engineering.py
        ```
    * **输入**: `processed_data/processed_data.csv`
    * **输出**: `features_engineered_data/features_engineered_data.csv` (包含原始列、`_clr` 列、单位协调后的列如 `_ppm`、比率列如 `_ratio`，以及对数变换后的列如 `_log`)

4.  **模型训练 (`src/model_training.py`)**
    * **目的**: 训练随机森林、SVM 和 DNN 模型。为随机森林执行树数量分析，为 DNN 执行超参数调优。保存训练好的模型、缩放器和标签编码器。
    * **特征选择**: 默认情况下，此脚本配置为使用 CLR 转换后的特征（`_clr` 后缀）。你可以在脚本的配置部分更改此设置。
    * **运行命令**:
        ```bash
        python src/model_training.py
        ```
    * **输入**: `features_engineered_data/features_engineered_data.csv`
    * **输出**:
        * 保存的模型: `model_outputs/tree_model_analysis/random_forest_model.joblib`, `model_outputs/svm_rbf_model.joblib`, `model_outputs/dnn_model_analysis/best_dnn_model.keras`
        * 保存的预处理器: `model_outputs/standard_scaler.joblib`, `model_outputs/label_encoder.joblib`
        * 随机森林树分析图: `model_outputs/tree_model_analysis/rf_logloss_vs_num_trees.png`
        * KerasTuner 日志: `model_outputs/dnn_model_analysis/keras_tuner_logs/`

5.  **模型评估 (`src/model_evaluation.py`)**
    * **目的**: 加载训练好的模型并在测试集上进行评估。生成混淆矩阵、ROC曲线、精确率-召回率曲线和特征重要性图。
    * **运行命令**:
        ```bash
        python src/model_evaluation.py
        ```
    * **输入**:
        * `features_engineered_data/features_engineered_data.csv` (用于重构测试集)
        * `model_outputs/` 目录下保存的模型和预处理器
    * **输出**: 评估图表保存在 `model_outputs/evaluation_plots/` 目录下。分类报告打印到控制台。

6.  **模型可解释性 (`src/model_interpretability.py`)**
    * **目的**: 使用 SHAP 解释模型预测。生成 SHAP 概要图 (蜂群图/条形图) 和依赖图。
    * **运行命令**:
        ```bash
        python src/model_interpretability.py
        ```
    * **输入**:
        * `features_engineered_data/features_engineered_data.csv`
        * `model_outputs/` 目录下保存的模型和预处理器
    * **输出**: SHAP 图表保存在 `model_outputs/interpretability_plots/` 目录下。关于如何进行地质意义讨论的指南打印到控制台。

## 6. 运行 Streamlit GUI 应用程序 (`app.py`)

Streamlit 应用程序提供了一个用户界面，用于上传新数据、选择已训练的模型、获取预测结果并可视化。

1.  **先决条件**: 确保所有依赖项、训练好的模型、缩放器和标签编码器都已按之前的设置和模型训练步骤准备就绪。
2.  **导航到项目根目录**: 打开终端（已激活 Conda 环境），并 `cd` 到项目根目录（`app.py` 所在的目录）。
3.  **运行应用程序**:
    ```bash
    streamlit run app.py
    ```
    你的网页浏览器应该会自动打开并导航到应用程序的 URL (通常是 `http://localhost:8501`)。

## 7. 使用 Docker 构建和运行 (可选加分项)

此步骤将 Streamlit 应用程序打包到 Docker 镜像中，以便于部署和移植。

1.  **先决条件**: Docker Desktop 已安装并正在运行。
2.  **确保文件就位**: 在你的项目根目录下，你需要：
    * `app.py`
    * `Dockerfile` (按照之前的说明提供)
    * `requirements.txt` (准确列出 `app.py` 的所有依赖项)
    * `.dockerignore` 文件
    * `model_outputs/` 目录，包含所有保存的模型和预处理器。
3.  **构建 Docker 镜像**:
    在终端中导航到项目根目录，并运行：
    ```bash
    docker build -t my_geochem_app .
    # 将 'my_geochem_app' 替换为你想要的镜像名称
    ```
    *此步骤可能需要稳定的互联网连接（如果你在中国大陆并遇到访问限制，可能需要在 Docker Desktop 中配置代理或 Docker Hub 镜像源，如之前讨论的那样）。*
4.  **运行 Docker 容器**:
    镜像成功构建后：
    ```bash
    docker run -p 8501:8501 my_geochem_app
    ```
    * `-p 8501:8501` 将你主机的 8501 端口映射到容器的 8501 端口。
5.  **访问应用程序**: 打开你的网页浏览器，访问 `http://localhost:8501`。

## 8. 项目模块涵盖的关键任务

1.  **数据提取与质量控制**: 读取 Excel, 数据类型/单位校验, 缺失值处理 (为CLR进行零值替换), 异常值标记, CLR转换。
2.  **探索性分析**: 成对散点图矩阵 (CLR), 相关性热图 (CLR), PCA双标图 (CLR), 地球化学比率图。
3.  **特征工程**: 单位协调 (例如 wt% 转 ppm), 对选定特征进行对数变换 (例如协调后的浓度、比率)。
4.  **模型训练**:
    * 随机森林: 误差与树数量关系图, 最优树数量确定。
    * SVM (RBF 或多项式核)。
    * DNN (Keras): 使用 KerasTuner 进行超参数搜索。
5.  **评估**: 混淆矩阵, ROC曲线及AUC, 精确率-召回率曲线 (均在测试集上), 非DNN模型的特征重要性条形图。
6.  **可解释性**: SHAP 概要图 (蜂群图/条形图), SHAP 依赖图, 关于有影响力变量及其地质意义的讨论指南。
7.  **可执行文件打包 (GUI)**: 独立的 GUI 程序 (Streamlit Web应用程序)，用于加载数据、选择模型、进行预测、保存结果和基本性能可视化。
8.  **Docker 打包 (可选)**: 将 Streamlit 应用程序打包到可工作的 Docker 镜像中。

## 9. 配置说明
* **文件路径**: 大多数脚本使用相对路径，并假设项目遵循标准结构。如果修改了结构，请相应更新脚本中的路径（尤其是在每个脚本的“路径配置”或“main”函数部分）。
* **训练特征选择**: `model_training.py` 脚本默认使用 CLR 转换后的特征（`_clr` 后缀）。你可以修改该脚本中的 `FEATURE_SUFFIX_USED_FOR_TRAINING` 变量来试验其他特征集（例如 `_ppm_log`）。评估和可解释性脚本需要与训练时使用的特征保持一致。
* **SHAP 的关键特征**: 在 `model_interpretability.py` 中，`CRITICAL_FEATURES_FOR_DEPENDENCE_PLOTS` 列表应根据 SHAP 概要图的结果或你的领域专业知识进行更新。
* **`app.py` 帮助部分的库版本**: 请记得用你工作环境中的实际版本更新 `app.py` “关于与帮助”部分中列出的占位库版本。

## 10. 常见问题故障排除
* **`ModuleNotFoundError`**: 确保你的 Conda 环境已激活，并且 `requirements.txt` 中的所有包都已安装在该环境中。
* **文件路径问题**: 如果脚本报告“文件未找到”，请仔细检查文件路径。
* **Docker 网络问题 (如果适用)**: 如果 `docker build` 无法拉取基础镜像，请确保 Docker Desktop 具有正确的网络/代理/镜像源设置，特别是当你在有网络访问限制的地区时。请参考我们之前关于此问题的讨论。
* **SHAP 性能**: 用于 SVM 的 `KernelExplainer` 可能运行缓慢。如果需要，请使用较小规模的背景数据集，或者对于 SVM 的全局特征重要性，可以更依赖于 `model_evaluation.py` 中计算出的排列重要性结果。
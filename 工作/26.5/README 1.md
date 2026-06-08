# Java 到仓颉转换与合并演示项目

本项目演示一个从 Java 程序迁移并增强既有仓颉程序的流程：先使用 `x2cangjie` 将 Java 科学计算器转换为仓颉骨架，再将补全后的新增功能合并到原仓颉计算器中，最终运行合并后的仓颉程序。

## 1. 项目目标

演示目标：

1. 展示一个既有仓颉计算器项目。
2. 展示一个功能更完整的 Java 科学计算器。
3. 使用 `x2cangjie` 真实生成仓颉骨架。
4. 使用合并脚本把 Java 侧新增功能合并到仓颉项目。
5. 编译并运行最终合并后的仓颉程序。

最终效果：

```text
原仓颉计算器：+、-、*、/
Java 科学计算器新增：%、^、max、min
最终仓颉计算器：+、-、*、/、%、^、max、min
```

## 2. 项目文件说明

```text
javaPJ/ScientificCalculator.java
```

Java 科学计算器源码，提供新增功能来源。

```text
cangjiePJ/test/src/main.cj
```

原始仓颉计算器源码，只支持四则运算。

```text
converted/ScientificCalculator.cj
```

由本地 `x2cangjie` 真实生成的仓颉骨架文件。

```text
converted/ScientificCalculator.completed.cj
```

补全函数体后的仓颉科学计算器版本，用于继续演示合并流程。

```text
cangjiePJ/merged/src/main.cj
```

最终合并后的仓颉程序。

```text
cangjiePJ/merged/cjpm.toml
```

最终合并程序的仓颉项目配置文件。

```text
tools/run_x2cangjie_scientific_calculator.py
```

运行本地 `x2cangjie`，从 Java 源码生成仓颉骨架。

```text
tools/merge_calculators.py
```

合并脚本，将补全后的仓颉科学计算器功能合并到原仓颉计算器。

```text
tools/run_merged_cangjie.sh
```

一键编译并运行最终合并后的仓颉程序。

## 3. 已完成工作

### 3.1 Java 科学计算器

完成文件：

```text
javaPJ/ScientificCalculator.java
```

实现内容：

- 支持命令行输入表达式。
- 输入格式为 `数字1 运算符 数字2`。
- 支持 `+`、`-`、`*`、`/`。
- 新增 `%`、`^`、`max`、`min`。
- 处理除零和取模除零错误。
- 对不支持的运算符给出错误提示。

### 3.2 原仓颉计算器

保留原仓颉项目作为合并目标：

```text
cangjiePJ/test/src/main.cj
```

原始版本只支持：

```text
+、-、*、/
```

### 3.3 x2cangjie 接入

本项目已接入本地 `x2cangjie`：

```text
third_party/x2cangjie/
```

当前已经跑通两个真实阶段：

```text
create_schema.py
create_skeleton.py
```

这说明 Java 源码可以被分析并转换为仓颉结构。当前骨架阶段生成的方法体仍为 `TODO`，完整函数体自动翻译需要继续配置大模型 API。

### 3.4 合并脚本

完成文件：

```text
tools/merge_calculators.py
```

合并脚本会执行以下操作：

- 读取原仓颉计算器。
- 读取补全后的仓颉科学计算器。
- 提取 `case` 运算分支。
- 识别原程序缺少的新运算符。
- 合并 `%`、`^`、`max`、`min`。
- 更新支持操作的提示文字。
- 加入 `%` 的除零保护。
- 为 `^` 运算追加 `power` 辅助函数。
- 输出最终合并文件。

### 3.5 最终仓颉程序运行

最终合并程序已整理为可运行仓颉项目：

```text
cangjiePJ/merged/
```

当前电脑的仓颉 SDK 路径：

```text
/Users/mj/cangjie
```

演示脚本：

```text
tools/run_merged_cangjie.sh
```

该脚本会自动设置仓颉 SDK 环境，执行 `cjpm build` 和 `cjpm run`。

## 4. 演示环境准备

进入项目根目录：

```bash
cd /Users/mj/Documents/work/26.5/java2cangjie
```

确认当前位置：

```bash
pwd
```

预期输出：

```text
/Users/mj/Documents/work/26.5/java2cangjie
```

## 5. 演示步骤

### 步骤 1：展示原仓颉计算器

查看原仓颉计算器：

```bash
sed -n '1,90p' cangjiePJ/test/src/main.cj
```

关注原始提示文字：

```text
支持的操作：+（加）、-（减）、*（乘）、/（除）
```

关注原始运算分支：

```text
case "+" => num1 + num2
case "-" => num1 - num2
case "*" => num1 * num2
case "/" => num1 / num2
```

演示说明：

```text
原仓颉计算器已有基础交互能力，但只支持加、减、乘、除四种运算。
```

### 步骤 2：展示 Java 科学计算器

查看 Java 科学计算器：

```bash
sed -n '1,140p' javaPJ/ScientificCalculator.java
```

关注新增运算：

```text
case "%":
case "^":
case "max":
case "min":
```

演示说明：

```text
Java 科学计算器在四则运算基础上增加了取模、幂运算、最大值和最小值。
这些新增功能是后续迁移并合并到仓颉程序中的来源。
```

### 步骤 3：运行 x2cangjie 生成仓颉骨架

执行：

```bash
python3 tools/run_x2cangjie_scientific_calculator.py
```

预期关键输出：

```text
Generated: data/java/skeletons/ScientificCalculator/src/ScientificCalculator.cj
Skeleton generation complete: data/java/skeletons/ScientificCalculator
x2cangjie 已生成: /Users/mj/Documents/work/26.5/java2cangjie/converted/ScientificCalculator.cj
注意: 当前脚本跑通的是 x2cangjie 的 schema + skeleton 阶段。
完整函数体翻译主要还需要 OPENAI/API 配置。
```

查看生成结果：

```bash
sed -n '1,80p' converted/ScientificCalculator.cj
```

预期可以看到仓颉类和方法结构：

```text
public open class ScientificCalculator
public static func calculate(...)
throw Exception('TODO')
```

演示说明：

```text
x2cangjie 已经将 Java 类和方法转换成仓颉结构。当前跑通的是 schema 和 skeleton 阶段，函数体自动翻译需要继续绑定大模型 API。
```

### 步骤 4：展示补全后的仓颉科学计算器

查看补全版本：

```bash
sed -n '1,130p' converted/ScientificCalculator.completed.cj
```

关注新增运算：

```text
case "%" => num1 - Float64(Int64(num1 / num2)) * num2
case "^" => power(num1, num2)
case "max" => if (num1 >= num2) { num1 } else { num2 }
case "min" => if (num1 <= num2) { num1 } else { num2 }
```

演示说明：

```text
该文件表示函数体补全后的仓颉科学计算器，用于继续演示合并流程。
```

### 步骤 5：运行合并脚本

执行：

```bash
python3 tools/merge_calculators.py cangjiePJ/test/src/main.cj converted/ScientificCalculator.completed.cj cangjiePJ/merged/src/main.cj
```

预期输出：

```text
已合并新增运算符: %, ^, max, min
输出文件: cangjiePJ/merged/src/main.cj
```

演示说明：

```text
合并脚本识别出原仓颉计算器缺少的四个运算符，并将它们合并到最终仓颉程序中。
```

### 步骤 6：展示最终合并结果

查看最终运算分支：

```bash
grep 'case "' cangjiePJ/merged/src/main.cj
```

预期输出：

```text
case "+" => num1 + num2
case "-" => num1 - num2
case "*" => num1 * num2
case "/" => num1 / num2
case "%" => num1 - Float64(Int64(num1 / num2)) * num2
case "^" => power(num1, num2)
case "max" => if (num1 >= num2) { num1 } else { num2 }
case "min" => if (num1 <= num2) { num1 } else { num2 }
```

演示说明：

```text
最终仓颉程序已经从支持 4 种运算扩展到支持 8 种运算。
```

### 步骤 7：运行最终仓颉程序

执行：

```bash
tools/run_merged_cangjie.sh
```

预期首先看到：

```text
cjpm build success
```

随后程序启动：

```text
欢迎使用仓颉计算器！
支持的操作：+（加）、-（减）、*（乘）、/（除）、%（取模）、^（幂）、max（最大值）、min（最小值）
请输入计算表达式:
```

依次输入以下表达式，每输入一行按一次回车：

```text
5 + 3
10 % 3
5 ^ 2
8 max 12
8 min 12
quit
```

预期结果：

```text
结果: 5.000000 + 3.000000 = 8.000000
结果: 10.000000 % 3.000000 = 1.000000
结果: 5.000000 ^ 2.000000 = 25.000000
结果: 8.000000 max 12.000000 = 12.000000
结果: 8.000000 min 12.000000 = 8.000000
```

演示说明：

```text
最终合并后的仓颉程序已经可以编译运行。原有四则运算保留，新增的取模、幂运算、最大值和最小值也可以正常执行。
```

## 6. 自动化运行演示

如果需要一次性展示所有运行结果，可以执行：

```bash
printf "5 + 3\n10 %% 3\n5 ^ 2\n8 max 12\n8 min 12\nquit\n" | tools/run_merged_cangjie.sh
```

该命令会自动输入测试数据，并输出所有计算结果。

## 7. 推荐演示顺序

1. 介绍项目目标：Java 功能迁移并增强仓颉程序。
2. 展示原仓颉计算器，只支持四则运算。
3. 展示 Java 科学计算器，新增 `%`、`^`、`max`、`min`。
4. 运行 `x2cangjie` 脚本，展示 Java 生成仓颉骨架。
5. 说明函数体自动翻译阶段需要大模型 API。
6. 展示补全后的仓颉科学计算器。
7. 运行合并脚本生成最终仓颉程序。
8. 展示最终仓颉代码支持 8 种运算。
9. 编译并运行最终仓颉程序。
10. 总结项目价值和后续扩展方向。

## 8. 演示总结

```text
本项目完成了一个 Java 到仓颉迁移和合并的完整演示。
首先保留一个已有仓颉计算器作为基础项目。
然后编写一个 Java 科学计算器，新增取模、幂运算、最大值和最小值。
接着接入本地 x2cangjie，真实跑通 Java schema 生成和仓颉 skeleton 生成。
由于完整函数体自动翻译需要绑定大模型 API，因此使用补全后的仓颉版本继续演示。
最后通过合并脚本把新增运算自动合并到原仓颉计算器中，并成功运行最终仓颉程序。
最终结果是：仓颉计算器从支持 4 种运算扩展到支持 8 种运算。
```

## 9. DeepSeek API Key 配置

DeepSeek 官方 API 兼容 OpenAI SDK，官方文档给出的 API 地址是：

```text
https://api.deepseek.com
```

本项目已在模型配置中加入 DeepSeek：

```text
third_party/x2cangjie/configs/model_configs.yaml
```

配置内容：

```yaml
models:
  deepseek-chat:
    model_id: deepseek-chat
    base_url: "https://api.deepseek.com"
    api_key: "${DEEPSEEK_API_KEY}"
    default_headers: {}
    total: 64000
    max_new_tokens: 8192
```

这里不会把真实 API key 写进项目文件，而是从环境变量 `DEEPSEEK_API_KEY` 读取。

### 9.1 临时绑定

临时绑定只在当前终端窗口有效：

```bash
export DEEPSEEK_API_KEY="sk-替换成真实DeepSeekKey"
```

### 9.2 永久绑定

如果是在个人电脑上使用，可以把环境变量写入 `~/.zshrc`。

执行下面命令时，需要把 `sk-替换成真实DeepSeekKey` 改成真实 DeepSeek API key：

```bash
printf '\n# DeepSeek API for x2cangjie\nexport DEEPSEEK_API_KEY="sk-替换成真实DeepSeekKey"\n' >> ~/.zshrc
```

然后让配置立即生效：

```bash
source ~/.zshrc
```

验证是否已经设置：

```bash
echo $DEEPSEEK_API_KEY
```

注意：演示或录屏时不要展示完整 API key。

### 9.3 使用 DeepSeek 尝试后续翻译

进入 `x2cangjie` 目录：

```bash
cd /Users/mj/Documents/work/26.5/java2cangjie/third_party/x2cangjie
```

尝试类型翻译：

```bash
bash scripts/java/translate_types.sh ScientificCalculator deepseek-chat 0.0 "" true false
```

尝试片段翻译：

```bash
bash scripts/java/translate_fragment.sh ScientificCalculator deepseek-chat "" 0.0 false
```

参数说明：

```text
deepseek-chat：使用 model_configs.yaml 中的 DeepSeek 模型配置。
0.0：温度参数，表示尽量稳定输出。
false：暂不启用 RAG，避免额外准备 CangjieCorpus 索引。
```

## 10. 当前限制与后续工作

当前已经可以完整演示：

- Java 科学计算器源码。
- x2cangjie 真实生成仓颉骨架。
- 补全后的仓颉科学计算器。
- 自动合并到原仓颉计算器。
- 最终仓颉程序编译运行。

后续可继续完善：

- 使用 DeepSeek API 尝试自动补全函数体翻译。
- 接入 RAG 语料和索引，提高复杂代码翻译质量。
- 将合并脚本从计算器专用扩展为更通用的仓颉源码合并工具。
- 增加更多测试案例。
- 增加 Java 结果与仓颉结果的自动对比。

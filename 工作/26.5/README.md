# Java 到仓颉转换与合并演示项目

本项目演示一个从 Java 程序迁移并增强既有仓颉程序的流程：先使用 `x2cangjie` 将 Java 科学计算器转换为仓颉骨架，再将补全后的新增功能合并到原仓颉计算器中，最终运行合并后的仓颉程序。

## 1. 项目目标

演示目标：

1. 展示一个既有仓颉计算器项目(翟）。
2. 展示一个功能更完整的 Java 科学计算器。
3. 使用 `x2cangjie` 真实生成仓颉骨架。
4. 使用合并脚本把 Java 侧新增功能合并到仓颉项目。
5. 编译并运行最终合并后的仓颉程序。


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


## 4. 演示步骤

### 步骤 1：展示原仓颉计算器

查看原仓颉计算器：

```bash
cangjiePJ/test/src/main.cj
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



原仓颉计算器已有基础交互能力，但只支持加、减、乘、除四种运算。

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



Java 科学计算器在四则运算基础上增加了取模、幂运算、最大值和最小值。
这些新增功能是后续迁移并合并到仓颉程序中的来源。


### 步骤 3：运行 x2cangjie 生成仓颉骨架

执行：

```bash
/usr/bin/python3 tools/run_x2cangjie_scientific_calculator.py
```


x2cangjie 已经将 Java 类和方法转换成仓颉结构。当前跑通的是 schema 和 skeleton 阶段，函数体自动翻译需要继续绑定大模型 API。

### 步骤 4：展示补全后的仓颉科学计算器

查看补全版本：

```bash
converted/ScientificCalculator.completed.cj
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



最终仓颉程序已经从支持 4 种运算扩展到支持 8 种运算。


### 步骤 7：运行最终仓颉程序

执行：

```bash
tools/run_merged_cangjie.sh
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




## 5. 当前限制与后续工作

当前已经可以完整演示：

- Java 科学计算器源码。
- x2cangjie 真实生成仓颉骨架。
- 补全后的仓颉科学计算器。
- 自动合并到原仓颉计算器。
- 最终仓颉程序编译运行。

后续可继续完善：

- 使用 DeepSeek API 尝试自动补全函数体翻译。
- 将合并脚本从计算器专用扩展为更通用的仓颉源码合并工具。


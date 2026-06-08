

**演示讲稿**

我演示的是一个 Java 到仓颉的代码迁移和功能合并流程。

这个项目的目标是：先从一个已有的仓颉计算器出发，再把 Java 科学计算器中的新增功能迁移过来，最后生成并运行一个功能更完整的仓颉程序。

[操作：进入项目目录，执行 `pwd`]

现在我已经进入项目目录，可以看到当前路径是 `java2cangjie`。

接下来先看原始的仓颉计算器。

[操作：执行 `sed -n '1,90p' cangjiePJ/test/src/main.cj`]

这里是原始仓颉程序。  
可以看到，它已经具备基本的输入、解析和计算流程，但是目前只支持四种基础运算：加、减、乘、除。

也就是说，当前仓颉计算器的能力比较基础，适合作为后续合并的目标项目。

接下来我们看 Java 版本的科学计算器。

[操作：执行 `sed -n '1,140p' javaPJ/ScientificCalculator.java`]

这里是 Java 源码。  
和刚才的仓颉版本相比，Java 版本在四则运算的基础上，额外增加了取模、幂运算、最大值和最小值。

也就是 `%`、`^`、`max` 和 `min` 这四个新增功能。

这些新增能力，就是我们后面要迁移并合并到仓颉项目里的内容。

为了证明 Java 程序本身是可运行的，我们先执行一下 Java 版本。

[操作：执行 `javac javaPJ/ScientificCalculator.java`]

现在先编译 Java 文件。

[操作：执行 Java 自动输入测试命令]

```bash
printf "5 + 3\n10 %% 3\n5 ^ 2\n8 max 12\n8 min 12\nquit\n" | java -cp javaPJ ScientificCalculator
```

可以看到，Java 程序能够正常输出结果。  
加法结果是 8，取模结果是 1，幂运算结果是 25，最大值是 12，最小值是 8。

这说明 Java 侧的新增功能是完整可用的。

接下来进入核心步骤：使用 x2cangjie 从 Java 源码生成仓颉骨架。

[操作：执行 `python tools/run_x2cangjie_scientific_calculator.py`]

这个脚本会调用本地的 x2cangjie 工具，对 Java 源码进行分析，并生成对应的仓颉结构。

现在可以看到，命令已经执行完成，并且生成了 `converted/ScientificCalculator.cj` 文件。

[操作：执行 `sed -n '1,90p' converted/ScientificCalculator.cj`]

这里展示的是 x2cangjie 生成的仓颉骨架。  
它已经把 Java 类和方法结构转换成了仓颉代码结构。

需要说明的是，当前演示跑通的是 schema 和 skeleton 阶段。  
也就是说，它已经完成了代码结构分析和骨架生成。  
完整的函数体我使用大模型进行补充
，这是补全函数体后的仓颉版本。

[操作：执行 `sed -n '1,130p' converted/ScientificCalculator.completed.cj`]

这个文件里已经包含了从 Java 科学计算器迁移过来的核心计算逻辑。  
可以看到，里面除了原来的加减乘除，还包含 `%`、`^`、`max` 和 `min`。

接下来，我们要把这些新增功能合并到原始仓颉计算器中。

[操作：执行合并命令]

```bash
python3 tools/merge_calculators.py cangjiePJ/test/src/main.cj converted/ScientificCalculator.completed.cj cangjiePJ/merged/src/main.cj
```

现在合并脚本已经执行完成。  
它识别出了原仓颉计算器中缺少的四个运算符：取模、幂运算、最大值和最小值，并把它们合并到了最终仓颉程序中。

接下来查看最终合并结果。

[操作：执行 `grep 'case "' cangjiePJ/merged/src/main.cj`]

这里可以看到，最终程序已经支持八种运算。  
前四个是原来仓颉计算器已有的加、减、乘、除。  
后四个是从 Java 科学计算器迁移过来的 `%`、`^`、`max` 和 `min`。

最后，我们编译并运行最终的仓颉程序。

[操作：执行最终运行命令]

```bash
printf "5 + 3\n10 %% 3\n5 ^ 2\n8 max 12\n8 min 12\nquit\n" | tools/run_merged_cangjie.sh
```

可以看到，仓颉项目已经成功编译。  
随后程序开始运行，并依次输出测试结果。

加法结果是 8。  
取模结果是 1。  
幂运算结果是 25。  
最大值结果是 12。  
最小值结果是 8。

这说明最终合并后的仓颉程序已经可以正常运行，新增功能也已经成功生效。

总结一下，本次演示完成了一个从 Java 到仓颉的迁移和合并流程。

首先，我们展示了一个只支持四则运算的原始仓颉计算器。  
然后，我们展示了一个功能更完整的 Java 科学计算器。  
接着，使用 x2cangjie 生成仓颉代码骨架。  
再通过补全后的仓颉代码和合并脚本，把 Java 侧新增功能合并到原仓颉项目中。  
最后，成功编译并运行最终的仓颉程序。

最终效果是：仓颉计算器从原来的四种基础运算，扩展到了八种运算。


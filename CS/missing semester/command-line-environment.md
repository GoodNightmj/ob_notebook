多数程序的参数由_标志_和普通字符串混合组成。标志可以通过其前面的短横线 ( `-` ) 或双短横线 ( `--` ) 来识别。标志通常是可选的，其作用是修改程序的行为。例如， `ls -l` 会改变 `ls` 命令的输出格式。


学习了解[glob模式](https://www.gulpjs.com.cn/docs/getting-started/explaining-globs)



## SSH
[阮一峰](https://wangdoc.com/ssh/basic)
[SSH Tutorial](https://zah.uni-heidelberg.de/it-guide/ssh-tutorial-linux)

## 常用shell 工具
[llm](https://github.com/simonw/llm)
tldr 


## HW
1. 你可能会看到类似 `cmd --flag -- --notaflag` 这样的命令。` `--` 是一个特殊参数，它告诉程序停止解析标志。` `--` 之后的所有内容都被视为位置参数。这有什么用呢？尝试运行 `touch -- -myfile` ，然后去掉 `--` 将其删除。

因为有些文件名或参数本身以 `-` 开头（比如 `-myfile`、`--help.bak`、`-rf` 等），而命令通常会把以 `-` 开头的字符串当作选项（option/flag）。这会导致歧义甚至危险操作。

```bash
touch -myfile
touch: illegal option -- y
usage: touch [-A [-][[hh]mm]SS] [-achm] [-r file] [-t [[CC]YY]MMDDhhmm[.SS]]
       [-d YYYY-MM-DDThh:mm:SS[.frac][tz]] file ...
❯ touch --  -myfile
❯ ls
-myfile
```
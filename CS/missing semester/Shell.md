可以说terminal 是shell的可视化界面，在shell中，bash是最常用的shell之一

shell 如何找到像 `date` 或 `echo` 的程序呢？如果 shell 被要求执行一个命令，它会查询一个名为 `$PATH` _环境变量_ ，该变量列出了 shell 在收到命令时应该搜索哪些目录来查找程序

当一个以 `#!/path` 开头的文件被执行时，shell 会启动 `/path` 处的程序，并将文件内容作为输入传递给它。对于 shell 脚本来说，这意味着将 shell 脚本的内容传递给 `/bin/bash`
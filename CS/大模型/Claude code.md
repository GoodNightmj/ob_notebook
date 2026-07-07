### Talk to your codebase：与代码库对话
使用@命令让claude 去看你具体的某个文件
@ 的用法有这几种：
- **@./src/App.tsx**：引用单个文件，Claude 只会读取这个文件的内容
- @./src/components/：引用整个目录，Claude 会拿到这个目录的文件列表
- **@./src/App.tsx @./src/styles/global.css**：引用多个文件，用空格隔开就行
这样能节省上下文空间


### Steer with modes：用模式驾驭 Claude
使用shift+tab切换模式。共有四种模式
四个模式对应四种信任级别。default 最稳、accept edits （即允许修改文件，但执行命令还需确认)
plan 复杂任务先规划
auto 长任务全自动。

### Undo anything：Claude 改错了？一键撤销
两种方式
- 连按两下esc
- 输入/rewind
需要注意的是，这两种方式回滚的不仅仅是文件内容，对话上下文也会一起回滚。
也就是说，回滚之后，Claude 会「忘记」那次操作之后发生的所有对话，就像那次操作从来没发生过一样。
但是如果执行了某些命令，比如下载了某些包，这种情况会滚是不能完成的

### Run in the background：让 Claude 在后台干活


### Teach Claude your rules：让 Claude 记住你的规则
CLAUDE.md 是给 Claude 装的「长期记忆」，写一次，永远记得。/init 快速创建，/memory 管理个人偏好，记得保持精简。

### 06｜Extend with tools：用 MCP 给 Claude 装外挂
MCP 全称是 Model Context Protocol，翻译过来叫「模型上下文协议」
可以把它理解为一个「外挂接口」。Claude Code 本身能做的事情是有限的：读写文件、执行命令、搜索代码。但如果你想让 Claude 做更多的事情呢？MCP可以把外部工具「接入」Claude Code，让 Claude 能调用这些工具的能力。
使用/mcp查看已经安装的外设


### Automate your workflow：让 Claude 自动化你的工作流
skill 和hook
Skills 给 Claude 装技能包，让它在特定领域更强；Hooks 给操作加钩子，实现自动化工作流。两个配合使用，Claude Code 直接变成你的定制化开发助手。

### Multiply yourself：让 Claude 的分身帮你干活
子代理的优势有
1. 独立视角。如果代码是 Claude 自己写的，你让它自己审查，它多少会「手下留情」。但如果你启动一个子代理来审查，这个子代理是一个全新的实例，它不知道这段代码是谁写的，只会客观地指出问题。
2. 保护主会话的上下文。子代理有自己独立的上下文空间，不管它读了多少文件、分析了多少代码，这些只占用它自己的上下文，对你的主会话零影响。等它干完活，只会把一份精炼的结果摘要汇报回来，占用很小。
使用/agents命令创建
### 常用命令
/model调节模型
/effort调节思考强度
/compact 压缩上下文
/clear彻底清空对话
/context 查看还有多少脑容量
claude --resume 进入之前的对话

### 会做检索增强生成：chunk、embed、retrieve、answer with citations。
chunk：切块
大模型不能每次把所有文档都读进去，所以要把文档切成小段。
embed：向量化
把文本变成一组数字，即向量
retrieve：检索
当用户提问时，也把问题变成向量，然后和所有 chunk 的向量计算相似度，找最相关的几个 chunk。
answer with citations：带引用回答
把检索到的 chunk 放进 prompt,让大模型只基于文档回答
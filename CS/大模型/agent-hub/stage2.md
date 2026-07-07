### 会做检索增强生成：chunk、embed、retrieve、answer with citations。
chunk：切块
大模型不能每次把所有文档都读进去，所以要把文档切成小段。
作用
1. 避免上下文太长  
2. 提高检索精度  
3. 方便后面做 citation
embed：向量化
把文本变成一组数字，即向量，使计算机可以用数学方式衡量两段文本的语义相似度。
retrieve：检索
当用户提问时，也把问题变成向量，然后和所有 chunk 的向量计算相似度，找最相关的几个 chunk。
answer with citations：带引用回答
把检索到的 chunk 放进 prompt,让大模型只基于文档回答


RAG=检索系统+生成模型
检索系统包括chunk，embedding，similar search 
生成模型包括prompt with context，answer with citations
也就是先去查资料，在让模型根据资料回答

代码大概分几个函数
```python
def load_documents(docs_dir:str) -> list[Dict[str, str]]:
去加载文档
def chunk_text(text: str, chunk_size: int =120,overlap: int = 30)-> list[str]:
去处理某个具体文档
def build_chunks(docs: list[Dict[str, str]])-> list[Dict[str, str]]:
去把所以文档都分好块
def normalize(vectors:np.ndarray)-> np.ndarray:
归一化，方便检测余弦相似度
def retrieve(query: str, chunks: list[Dict[str, str]],chunk_embeddings: np.ndarray, embedder: SentenceTransformer, top_k: int = 2) -> list[Dict[str, str]]:
去将问题和资料做检索
def build_context(retrieved_chunks: list[Dict[str, str]]) -> str:
将相关的资料放到提示词中
def answer_with_citations(query: str, retrieved_chunks: list[Dict[str, str]]) -> str:
做回答
```


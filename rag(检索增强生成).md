## rag(检索增强生成)

模型幻觉问题、时效性问题、数据安全问题

知识库->（提取文本）->字符串->（划分段落区域）->文本区块chunk->(文本嵌入)->向量库

用户输入一问题->(文本嵌入)->问题文本的嵌入向量->相似匹配->向量库

向量库->prompt->LLM->基于知识库问答对话ai

KBQA(基于知识库)<主题，关系，对象>

rag相对于之前的，引入外部知识库的检索机制
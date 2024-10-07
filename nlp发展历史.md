# NLP发展历史

**全监督学习阶段(2017)**

  前神经网络时代（svm；专家系统；决策树：特征工程，利用自己的知识手动提取简明特征）：n-gram模型

  神经网络时代（lstm；seq2seql：结构工程提取特征工作交给神经网络隐层，包括rnn，出现了利用语言模型和序列自编码器进行半监督学习，到了预训练时代，半监督学习逐渐演变成了自监督学习）：LSTM

##### 

**预训练+fine-tuning（2017.6 transformer诞生）**

  放弃RNN结构，采用位置编码与自注意力机制，并且设计了多头注意力机制，在并行上低于cnn而远高于rnn，信息柔和能力上低于rnn而高于cnn

**（2017.2）语境化词嵌入**

过渡阶段transition：结构工程，基于特征的预训练凡是，双向LSTM

**预训练+fine-tune（2017-2019）**

decoder-Only：基于大量的无监督语料去利用预训练任务，而后针对具体下游人物的目标函数做fine-tune，选对一个好的目标函数可以让预训练效果大增

gpt只是一个有解码器的transformer，利用传统的下一词语言预测模型

Encoder-only：bert是一个只有编码器的transformer，编码器可以注意到全部时间步的信息，预训练任务为MLM，基于上下文。

Prefix LM：UniLM

Encoder-Decoder：T5

  预训练+微调阶段，architecture engineering逐渐消失

![](C:\Users\zyr\Pictures\Screenshots\屏幕截图 2024-10-07 235140.png)

**Zero-shot（2019.2）**

模型越来越大，微调逐渐变得贵不可能，放弃微调范式，在预训练做下游任务时，并不需要任何额外的标注信息，也不去改模型参数，只证明：模型增大，性能增长：GPT-2

**预训练+prompt+predict**

**In-context Learning上下文学习**：提示工程prompt engineering不去调整模型权重，而是在输入窗口给予样本：GPT-3

**Instruction-tuning基于指令微调**

泛化能力强，实现价值观对齐，让模型通过压缩输出想要的内容：FLANInstructGPT

**chain of thoughts思维链**

特殊的In-Context Learning，需要在输入和输出之间给出一个推理出答案的过程，成为relationale（PaLm）


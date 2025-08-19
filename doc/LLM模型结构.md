# LLM模型结构

大模型(LLM)通常基于Transformer架构，主要以下部分组成：
- 输入嵌入层(Embedding Layer): 把词/子词转换为向量表示；常用词嵌入 + 位置编码(Positional Encoding/RoPE)
- Transformer堆叠层: 每一层由(自注意力机制(Self-Attention)、前馈网络(Feed-Forward Network,FFN)、归一化(LayerNorm/RMSNorm)、残差连接(Residual Connection))
- 输出层: 将Transformer的隐状态投影回词表维度，使用Softmax预测下一个token的概率分布

## LLM 输入层

### 作用/功能

1. 将离散的符号(token, 字, 词, 子词, 字节)转换为连续的向量，便于模型计算
2. 引入位置信息, 因为Transformer自身对序列位置不敏感
3. 可加入额外的条件信息, 如段落ID、模态标识(图像/文本)、任务标识(翻译/摘要)

### 主要技术与算法

#### 词/子词嵌入(Token Embedding)

- One-hot -> 词向量映射(典型: Word2Vec, Glove), 将词表中的每个token映射到一个稠密向量
- 子词切分(Subword Tokenization)
  - BPE(Byte-Pair Encoding): GPT、LLaMA使用
  - WordPiece: BERT使用
  - SentencePiece: T5、GLM使用
  - UnigramLM: 一些日文/多语言模型常用
- 字节级(Byte-level)嵌入, GPT-2使用它，可以统一处理多语言和符号

优点:
    1. 子词方法: 减少词表大小、处理OOV(未登录词)
    2. 字节方法: 完全无OOV, 跨语言泛化强

#### 位置编码(Positional Encoding)

Transformer没有循环结构, 需要额外注入位置信息

常见方法：
- 绝对位置编码(Absolute Positional Encoding): 
  - 正弦位置编码(sin/cos, 原始Transformer使用)
  - 学习型位置向量(Learnable Positional Embedding, BERT使用)
  - 优点: 简单直观，易实现
  - 缺点: 序列长度固定, 泛化到更长序列时性能下降
- 相对位置编码(Relative Positional Encoding, RPE)
  - Transformer-XL、DeBERTa使用
  - 位置是相对差值, 能更好的捕捉局部依赖
  - 优点: 支持长序列, 泛化好
  - 缺点: 实现复杂、计算开销稍大
- 旋转位置编码(RoPE, Rotary Positional Embedding)
  - GPT-NeoX、LLaMA系列使用
  - 通过在复数平面旋转向量引入位置信息
  - 优点: 更自然的扩展到长序列，效果优于RPE
- ALiBi(Attention with Linear Biases)
  - MPT、Falcon使用
  - 在注意力分数中直接引入线性位置偏置
  - 优点: 可无限扩展上下文, 不需要额外位置嵌入矩阵
- 多模态输入嵌入(扩展趋势)
  - 图像: ViT Patch Embedding(将图片切patch, 再嵌入)
  - 音频: Mel-Spectrogram -> Transformer输入
  - 代码/任务表示: 特殊token作为提示

对比和优缺点总结:
|技术|应用模型|优点|缺点|
|:--|:-----|:--|:--|
|BPE/子词嵌入|GPT, LLaMA|减少词表大小, 泛化较好|中文分词不自然|
|字节级嵌入|GPT-2|无OOV, 支持任务符号|序列更长、效率低|
|绝对位置编码|BERT|简单直观|长序列扩展差|
|相对位置编码(RPE)|Transformer-XL, DeBERTa|长文本泛化好|复杂度高|
|RoPE|LLaMA,GPT-NeoX|长文本效果好, 训练稳定|超长序列仍有限制|
|ALiBi|MPT, Falcon|可无限上下文, 计算简单|精度略低于RoPE|
|多模态嵌入|Gemini,GPT4-4o|支持文本+图像+音频|需要额外适配模块|
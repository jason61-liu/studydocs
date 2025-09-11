## DeepSeek-V3
- MLA,MHA
- KV Cache：Attention机制进阶的核心
	- 应用于推理阶段
	- 只存在于解码器
	- 加速QKV的矩阵相乘
	- KVCache会增加内存占用
- MQA：Multi-Query Attention:多头共享同一组key和value，存在精度损失
	- 降低计算复杂度
	- 更低内存使用率
	- 保持性能
- GQA：Group Query Attenttion：对key和value进行分组，降低精度损失
	- 分组查询
	- 共享key+value表示
	- 高效计算
- MLA：Multi Head Latent Attention 多头潜在注意力机制
	- 低秩联合压缩/低秩投影，key+value，减小kvcache
	- 缓存维度比较低的向量（位置向量+内容向量）进一步降低kvcache缓存
	- 同时对Q向量也采用和kv的处理方式一样，也是一个低维度
- 结构图
	![](attachments/Pasted%20image%2020250903001706.png)
- MOE（混合专家模型）-稀疏模型，简化计算量
	- 稠密模型：当模型计算时，所有参数都会参与计算
	- 稀疏模型：当模型计算时，部分参数激活计算，并非全部参数参与计算
	- 共享专家+路由专家（Top K）
		- 路由专家使用负载不均衡
		- 偏置，辅助损失，动态调整
- MTP（Multi-Token Prediction）
	![](attachments/Pasted%20image%2020250903003853.png)
	- 预测未来token
	- 超参数（并行生成的token个数），交叉熵
	- 将一个token生成转换为多个token，应用在训练和推理阶段
## DeepSeek- R1
- deepseek-r1-zero  RL方式不使用SFT
- deepseek-r1 基于deepseek-v3模型
- AGI通用人工智能
- code-start data
- 通过R1有训练出6个小模型
- 有监督学习SFT对于有见过数据表现好，没见过数据数据表现差
- RL强化学习在有见过和没有见过数据表象都很好，说明泛化能力强
- DeepMind-->RL
- OpenAI-->RL
----
- R1-zero PureRL--LLM--COT
- R1 SFT+RL--LLM--COT
- 蒸馏
- reward hacking

---
R! 直接通过RL训练，不使用SFT
- major vocting
- sft initialization
- rejection samping
- 蒙特卡洛树搜索
GRPO VS PPO
![](attachments/Pasted%20image%2020250906155413.png)- 671B参数，激活37B参数 训练采用 H800进行训练
- 上下文窗口扩展至32K和128K
- 使用F8混合精读训练得到的大号MoE模型
- 并行策略
	- 64路专家并行，16路流水线并行，数据并行--训练
- 指标：
	- TFTF 首token生成时间，衡量prefill
	- TPOT 生成每个token时间，衡量 decode
- 特点
	 - Prefill：计算密集型，完成kv cache的生成后， 本身无需继续保留这些缓存
	 - Decode：访存密集型，需要尽可能多地保留缓存数据以及保障推理效率
- Attention:
	- TP=4 DP=8
	- MOE EP=32
	- 冗余专家 数量动态调整 prefill阶段 32个冗余阶段
- device-limited Routing:每个token最多放到Mge设备，首先选择最高得分专家所在的M个设备，进而降低通信成本
----
主要创新点
- MLA 多头潜在注意力机制
- DeepseekMoE
- 多token预测
- FP8训练混合精度训练：节约内存，减少通信传输，计算速度快
llama 如何优化注意力机制的计算的
- 注意力机制 MHA-->GQA 减少kv cache 对显存的占用，保证性能不受大的影响
- 词汇表  预训练语料，训练成本
- 绝对位置编码（三角函数编码）

Transformer 为什么要用Laynormal层归一化
- 模型训练的稳定性
- 归一化可以加速模型的收敛
- 模型训练不依赖weight的初始化
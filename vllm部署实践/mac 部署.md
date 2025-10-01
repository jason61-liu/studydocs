- 下载vllm
```
git clone [https://github.com/vllm-project/vllm.git](https://github.com/vllm-project/vllm.git "https://github.com/vllm-project/vllm.git")
```
- 创建虚拟环境
```
uv venv -p 3.12 .vllm
source .vllm/bin/activate
```
- 编译vllm
```
cd vllm
uv pip install -r requirements/cpu.txt
uv pip install -e .
```
- 运行
```
vllm serve --max_model_len 20480 --max_num_batched_tokens 65536 --gpu_memory_utilization 0.95 ./models/Qwen3-0.6B
```
![](attachments/Pasted%20image%2020251001103610.png)
- 请求
```
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "./models/Qwen3-0.6B",   
    "messages": [
    {"role": "system", "content": "你是个友善的AI助手。"},
    {"role": "user", "content": "介绍一下什么是大模型推理。" }
    ]}'
```
![](attachments/Pasted%20image%2020251001103717.png)
查看后台
![](attachments/Pasted%20image%2020251001103800.png)

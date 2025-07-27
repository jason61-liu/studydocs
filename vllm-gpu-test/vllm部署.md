
- 租用实例
![](attachments/Pasted%20image%2020250727121857.png)
- 当前环境
![](attachments/Pasted%20image%2020250727113454.png)
- 查看cuda版本
![](attachments/Pasted%20image%2020250727113531.png)
- 下载模型
```shell
 huggingface-cli download --resume-download Qwen/Qwen3-0.6B --local-dir Qwen3-0.6B

huggingface-cli download --resume-download Qwen/Qwen2.5-1.5B-Instruct --local-dir Qwen2.5-1.5B-Instruct
```

- 配置环境
```
# 创建uv虚拟python环境
uv venv --python 3.12 --seed 
# 激活环境
source .venv/bin/activate
```
- 运行vllm
```shell
# 下载安装vllm
uv pip install vllm -i https://pypi.tuna.tsinghua.edu.cn/simple
# 启动vllm
vllm serve Qwen/Qwen3-0.6B
```

![](attachments/Pasted%20image%2020250727114015.png)
- 测试
```shell
curl http://localhost:8000/v1/models
```
![](attachments/Pasted%20image%2020250727114337.png)
```python
curl http://localhost:8000/v1/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen3-0.6B",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
    }'
```
![](attachments/Pasted%20image%2020250727114637.png)
- 源码编译方式启动vllm
```shell
git clone https://github.com/vllm-project/vllm.git
cd vllm
python use_existing_torch.py
pip install -r requirements/build.txt

# 设置一下，避免被oom
export MAX_JOBS=1

pip install --no-build-isolation -e .

```
- 开始编译
![](attachments/Pasted%20image%2020250727120310.png)
- 编译成功
![](attachments/Pasted%20image%2020250727133931.png)
- 运行
```shell
python -m vllm.entrypoints.openai.api_server \
    --host 0.0.0.0 \
    --port 8001 \
    --model Qwen/Qwen2.5-1.5B-Instruct \
    --trust-remote-code
```
![](attachments/Pasted%20image%2020250727134337.png)
![](attachments/Pasted%20image%2020250727134413.png)
- 测试
```shell
curl http://localhost:8001/v1/models
```
![](attachments/Pasted%20image%2020250727134555.png)
```shell
curl http://localhost:8001/v1/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen2.5-1.5B-Instruct",
        "prompt": "Beijing is a",
        "max_tokens": 7,
        "temperature": 0
    }'
```
![](attachments/Pasted%20image%2020250727134740.png)
- vllm server日志打印
![](attachments/Pasted%20image%2020250727135321.png)
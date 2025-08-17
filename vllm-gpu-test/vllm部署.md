
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
# 3.docker部署vllm
- 搭建基础环境
![](attachments/Pasted%20image%2020250803102931.png)
- 拉取镜像
![](attachments/Pasted%20image%2020250803103021.png)
- 启动容器
```shell
docker run --runtime nvidia --gpus all -v /root/Qwen2.5-7B-Instruct:/root/Qwen2.5-7B-Instruct -p 8000:8000 --ipc=host vllm/vllm-openai:latest --model /root/Qwen2.5-7B-Instruct --trust-remote-code
```
![](attachments/Pasted%20image%2020250803153108.png)
![](attachments/Pasted%20image%2020250803152727.png)
![](attachments/Pasted%20image%2020250803152512.png)
- 测试
```shell
curl http://8.130.172.187:8001/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "/root/Qwen2.5-7B-Instruct",   
    "messages": [
    {"role": "system", "content": "你是个友善的AI助手。"},
    {"role": "user", "content": "介绍一下什么是大模型推理。" }
    ]}'
```
![](attachments/Pasted%20image%2020250803152851.png)
![](attachments/Pasted%20image%2020250803152952.png)

```shell
docker stop benchmark;docker rm benchmark;docker run -itd --rm --runtime nvidia --gpus all --name benchmark -v /root:/root --net=host -m 100G --cpus 20 --entrypoint bash vllm/vllm-openai:latest


docker run --runtime nvidia --gpus all -v /root/Qwen2.5-7B-Instruct:/root/Qwen2.5-7B-Instruct -p 8000:8000 --ipc=host vllm/vllm-openai:latest --model /root/Qwen2.5-7B-Instruct --trust-remote-code



export LMCACHE_USE_EXPERIMENTAL=True \ 
export LMCACHE_CHUNK_SIZE=256 \ 
export LMCACHE_LOCAL_DISK="file://root/cachedata" \ 
export LMCACHE_MAX_LOCAL_DISK_SIZE=5.0 \ 
export LMCACHE_USE_EXPERIMENTAL=True \ 
python3 -m vllm.entrypoints.openai.api_server \
    --host 0.0.0.0 \
    --port 8001 \
    --model /root/Qwen2.5-7B-Instruct \
    --trust-remote-code \
    --swap-space "0" \
    --log-level debug \
    --kv-transfer-config '{"kv_connector":"LMCacheConnectorV1", "kv_role":"kv_both"}'
```
- lmcahe
```shell
env LMCACHE_USE_EXPERIMENTAL=True LMCACHE_CHUNK_SIZE=256 LMCACHE_LOCAL_DISK="file:///root/cachedata" LMCACHE_MAX_LOCAL_DISK_SIZE=5.0 python3 -m vllm.entrypoints.openai.api_server     --host 0.0.0.0     --port 8001     --model /root/Qwen2.5-7B-Instruct     --trust-remote-code     --swap-space "0"     --kv-transfer-config '{"kv_connector":"LMCacheConnectorV1", "kv_role":"kv_both"}
```
![](attachments/Pasted%20image%2020250817104711.png)
请求
发送请求
```shell
curl http://8.130.172.187:8001/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "X-Session-ID: user_123_conversation" \
    -d '{
    "model": "/root/Qwen2.5-7B-Instruct",   
    "messages": [
    {"role": "system", "content": "你是个友善的AI助手。"},
    {"role": "user", "content": "介绍一下什么是大模型推理。" }
    ]}'
```
![](attachments/Pasted%20image%2020250817105155.png)
查看后台日志
![](attachments/Pasted%20image%2020250817105104.png)

- 租用实例
![](attachments/Pasted%20image%2020250727121857.png)
- 当前环境
![](attachments/Pasted%20image%2020250727113454.png)
- 查看cuda版本
![](attachments/Pasted%20image%2020250727113531.png)
- 配置环境
```
# 创建uv虚拟python环境
uv venv --python 3.12 --seed 
# 激活环境
source .venv/bin/activate
# 下载安装vllm
uv pip install vllm -i https://pypi.tuna.tsinghua.edu.cn/simple
# 下载模型库
 huggingface-cli download --resume-download Qwen/Qwen3-0.6B --local-dir Qwen3-0.6B

huggingface-cli download --resume-download Qwen/Qwen2.5-1.5B-Instruct --local-dir Qwen2.5-1.5B-Instruct

# 运行vllm
vllm serve Qwen/Qwen3-0.6B
```
- 自测
```shell
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
- 源码编译vll
```shell
git clone https://github.com/vllm-project/vllm.git
cd vllm
python use_existing_torch.py
pip install -r requirements/build.txt

# 设置一下，避免被oom
export MAX_JOBS=1

pip install --no-build-isolation -e .

```
开始编译
![](attachments/Pasted%20image%2020250727120310.png)
编译成功
![](attachments/Pasted%20image%2020250727123101.png)
# 1.裸机部署
![](attachments/Pasted%20image%2020250802180923.png)
 - 购买云服务器
![](attachments/Pasted%20image%2020250802160714.png)
- 下载驱动https://www.nvidia.cn/drivers/lookup/
   ![](attachments/Pasted%20image%2020250802160842.png)
   - 安装**仓库配置包**驱动
   ```shell
# 查询
dpkg -l|grep nvidia
#卸载
sudo apt purge nvidia-driver-local-repo-ubuntu2204-570.172.08

apt install ./nvidia-driver-local-repo-ubuntu2204-570.172.08_1.0-1_amd64.deb
``` 
![](attachments/Pasted%20image%2020250802161901.png)
![](attachments/Pasted%20image%2020250802172713.png)
- 安装 NVIDIA 驱动 570 版本```
 ```shell
sudo apt install nvidia-driver-570 
```
![](attachments/Pasted%20image%2020250802180959.png)
- 安装nvidia-cuda-toolki thttps://developer.nvidia.com/cuda-toolkit-archive
![](attachments/Pasted%20image%2020250802181601.png)
```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.debsudo 
dpkg -i cuda-keyring_1.1-1_all.debsudo 
apt-get updatesudo 
apt-get -y install cuda-toolkit-12-8


#安装完成后配置环境变量
export PATH=/usr/local/cuda-12.8/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.8/lib64:$LD_LIBRARY_PATH

```
![](attachments/Pasted%20image%2020250802183439.png)
- 测试
```python
import torch

def check_cuda_with_pytorch():
    """检查 PyTorch CUDA 环境是否正常工作"""
    try:
        print("检查 PyTorch CUDA 环境:")
        if torch.cuda.is_available():
            print(f"CUDA 设备可用，当前 CUDA 版本是: {torch.version.cuda}")
            print(f"PyTorch 版本是: {torch.__version__}")
            print(f"检测到 {torch.cuda.device_count()} 个 CUDA 设备。")
            for i in range(torch.cuda.device_count()):
                print(f"设备 {i}: {torch.cuda.get_device_name(i)}")
                print(f"设备 {i} 的显存总量: {torch.cuda.get_device_properties(i).total_memory / (1024 ** 3):.2f} GB")
                print(f"设备 {i} 的显存当前使用量: {torch.cuda.memory_allocated(i) / (1024 ** 3):.2f} GB")
                print(f"设备 {i} 的显存最大使用量: {torch.cuda.memory_reserved(i) / (1024 ** 3):.2f} GB")
        else:
            print("CUDA 设备不可用。")
    except Exception as e:
        print(f"检查 PyTorch CUDA 环境时出现错误: {e}")

if __name__ == "__main__":
    check_cuda_with_pytorch()

```
![](attachments/Pasted%20image%2020250802184125.png)
# 2.docker

![](attachments/Pasted%20image%2020250802193618.png)
![](attachments/Pasted%20image%2020250802194846.png)
- 安装docker 
https://blog.csdn.net/weixin_44355653/article/details/140267707
![](attachments/Pasted%20image%2020250802191758.png)
- nvidia-container-toolkit 
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
```shell
#Configure the production repository:
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    
#Optionally, configure the repository to use experimental packages:
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list

#Update the packages list from the repository:
Update the packages list from the repository:
sudo apt-get update

#Install the NVIDIA Container Toolkit packages:
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
  sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```
- 配置使用该 runtime
```shell
#Configure the container runtime by using the `nvidia-ctk` command:
sudo nvidia-ctk runtime configure --runtime=docker
#The `nvidia-ctk` command modifies the `/etc/docker/daemon.json` file on the host. The file is updated so that Docker can use the NVIDIA Container Runtime.


#Restart the Docker daemon:
sudo systemctl restart docker
```
- 测试
```shell
 docker run --rm --gpus all  nvidia/cuda:12.0.1-runtime-ubuntu22.04 nvidia-smi
#`--gpu` 参数可选值：

# `--gpus all`：表示将所有 GPU 都分配给该容器
# `--gpus "device=<id>[,<id>...]"`：对于多 GPU 场景，可以通过 id 指定分配给容器的 GPU，# 例如 –gpu “device=0” 表示只分配 0 号 GPU 给该容器GPU 编号则是通过`nvidia-smi` 命令进行查看
```
![](attachments/Pasted%20image%2020250802192910.png)
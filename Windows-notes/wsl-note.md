
## WSL 备份

使用 `wsl --export` 命令导出镜像，参数依次是：子系统名称、备份路径（一般为tar文件）

```shell
wsl --export Ubuntu-24.04 Ubuntu2404.tar
wsl --export Ubuntu-24.04 e:\wsl\Ubuntu2404.tar
```

## WSL 还原

使用 `wsl --import` 命令还原镜像，参数依次是：子系统名称、安装文件夹、镜像文件路径

```shell
wsl --import Ubuntu-24.04 e:\wsl e:\wsl\Ubuntu2404.tar
```

## WSL 自动调整磁盘空间

> 该小节部分内容引用[V2EX网友的贴文](https://v2ex.com/t/975098)

关于自动释放 WSL2 虚拟硬盘空间，需要设置稀疏 VHD。

首先在上面的配置里再加一行：

```ini
sparseVhd=true
```

然后运行这个命令切换到稀疏 VHD：`wsl --manage 发行版名字 --set-sparse true`

比如 `wsl --manage Ubuntu --set-sparse true`

同时，设置 WSL 的配置文件，增加：

```ini
[experimental]
autoMemoryReclaim=gradual # 可以在 gradual 、dropcache 、disabled 之间选择
```

**注意：** 如果你在 WSL 里使用 docker，那需要将 `autoMemoryReclaim` 配置为 `dropcache` 或者 `disabled`，然后在 `/etc/docker/daemon.json` 里添加一句 `"iptables": false` ，否则你可能无法正常使用 docker。

## WSL Docker 中调用宿主机显卡

> 本小节提到的显卡是 Nvidia

- 首先去[官网](https://www.nvidia.com/en-us/drivers/)更新驱动并启动 CUDA。

- 在 WSL 中运行以下命令安装 `nvidia-container-toolkit`：

```sh
  sudo apt-get update
  sudo apt-get install -y nvidia-container-toolkit
```

- 在 WSL 中验证 GPU 的可用性：

```
  nvidia-smi
```

如果能显示类似下面的内容，则配置成功：

```text
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.02              Driver Version: 560.94         CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 ...    On  |   00000000:01:00.0 Off |                  N/A |
| N/A   49C    P8              2W /  125W |    1269MiB /   8188MiB |      2%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

接着，在 Docker 中启用 GPU 的支持

- 确保 Docker Desktop 已启用 WSL 2 集成：
    
    - 打开 Docker Desktop 设置。

    - 进入 **Settings > Resources > WSL Integration**，勾选你的 WSL 2 发行版。

- 启用 GPU 支持： 编辑 Docker 的 `daemon.json` 文件，添加以下内容：

```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```


然后重启 Docker 服务。

然后运行以下命令检查 Docker 是否能识别 GPU：

```sh
docker run --rm --gpus all nvidia/cuda:12.6.3-cudnn-runtime-ubuntu24.04 nvidia-smi
```

如果提示：`Unable to find image` 错误，则需要先去[这里](https://hub.docker.com/r/nvidia/cuda/tags)找到适合你的系统的镜像，拉取下来。拉取到镜像后，将你拉取的镜像名替换掉上面命令的 `nvidia/cuda:12.6.3-cudnn-runtime-ubuntu24.04` 这部分，然后重新运行命令。
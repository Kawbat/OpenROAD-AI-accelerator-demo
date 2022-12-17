本文档介绍了笔者构建OpenROAD工具链的全流程。

详情参见OpenROAD官方文档，包括但不限于：

[Build from sources using Docker — OpenROAD documentation](https://openroad.readthedocs.io/en/latest/user/BuildWithDocker.html)

首先，在命令行中安装docker：

```bash
sudo apt-get install docker.io
```

开启docker服务：

```bash
sudo service docker start
```

测试docker是否安装成功，在命令行中输入：

```bash
sudo docker version
```

注意，docker需要权限来连接/var/run/docker.sock。

显示以下信息：

```bash
Client:
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.2
 Git commit:        20.10.12-0ubuntu2~18.04.1
 Built:             Fri Oct 21 08:25:48 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server:
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.2
  Git commit:       20.10.12-0ubuntu2~18.04.1
  Built:            Mon Apr  4 20:53:56 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.9-0ubuntu1~18.04.2
  GitCommit:        
 runc:
  Version:          1.1.0-0ubuntu1~18.04.1
  GitCommit:        
 docker-init:
  Version:          0.19.0
  GitCommit:
```

测试docker提供的hello_world，在命令行中输入：

```bash
docker run hello-world
```

提示以下信息，说明docker可以正常运行：

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

接下来获取OpenROAD源代码。

如果虚拟机上未安装git，可以通过：

```bash
sudo apt install git
```

安装git。

接下来克隆OpenROAD源代码：

```bash
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git
```

也可以通过其他方式下载源代码（例如直接下载压缩包）。

源文件体积较大，下载需要一定时间。如果因大陆地区网络环境无法进行下载，在此不便展开讲解，可以考虑通过镜像源下载。

接下来开启docker：

注意，如果之前使用snap进行安装，需要通过

```bash
sudo snap start docker
```

启动docker。否则使用：

```bash
systemctl start docker
```

启动成功后无提示，或者提示：

```bash
Started.
```

安装OpenROAD依赖：

OpenROAD提供了一个脚本etc/DependencyInstaller.sh，支持Centos7 and Ubuntu 20.04。

要使用这个脚本安装依赖，需要使用最高权限。

```bash
sudo ./etc/DependencyInstaller.sh -run
sudo ./etc/DependencyInstaller.sh -dev
```

默认的构建模式是release，编译产生优化后的代码。产生的可执行文件位于build/src/openroad。

使用提供的脚本进行构建：

```bash
dos2Unix ./etc/Build.sh
./etc/Build.sh
```

由于OpenROAD使用cmake进行构建，需要安装cmake。同样，环境中需要有编译工具（g++）。安装方式不再赘述。

也可以手动编译：

```bash
mkdir build
cd build
cmake ..
make
```

第二种方式是使用docker进行构建。为了保证构建过程中的线程数量不发生冲突，

首先查看可用处理器核心数量N：

```bash
docker run centos:centos7 nproc
```

在运行构建脚本时将附带此参数。

获取OpenROAD脚本并构建工具链：

```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts
sudo ./build_openroad.sh --threads N
```

将使用docker进行构建，创建一个docker镜像。

成功使用docker辅助构建OpenROAD：

![HU94]1E$C@VXAB_Z6~SLMB6.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/903e3adf-78d8-461c-b2c4-398c83425340/HU941ECVXAB_Z6SLMB6.png)

接下来验证安装是否成功。

二进制工具链存放在Docker容器中。运行一个docker容器：

```cpp
sudo docker run -it -u $(id -u ${USER}):$(id -g ${USER}) -v $(pwd)/flow/platforms:/OpenROAD-flow-scripts/flow/platforms:ro openroad/flow-scripts
```

进入容器的命令终端后，测试工具链：

```cpp
source ./setup_env.sh
yosys -help
openroad -help
cd flow
make
exit
```

make后显示产生了gds文件，说明工具链构建成功：

![IW37L`BP~@N_J2TN]]MQ7]G.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/84d19644-4b89-4610-9aa4-6ff1750eb916/IW37LBPN_J2TNMQ7G.png)
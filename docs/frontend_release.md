# 挣钱宝前端安装部署指南

> 负责人：邱奕浩
> 审核人：邱晓裕

挣钱宝是一个用于发布有偿问卷的工具，采用`react`框架实现。本文件将一步步地给出应用的本地安装方法和云服务器部署的详细。



## 安装步骤

> 本安装教程支持 `windows` 系统，`linux` 系统未测试。

以下命令的目的是在本地运行`react`应用。

* **安装`node.js`**（若已安装，则可跳过）

登录 `node.js` [官网](<http://nodejs.cn/download/>)，下载 `node.js` 安装包进行本地安装。

目前测试过的最低版本为`v.8.11.3` ，建议下载`10.16.0`。

* **下载代码包**

```
git clone git@github.com:software-system-analysis-and-design/zhengqianbao_frontend.git
cd zhengqianbao_frontend
```

* **安装必要的依赖**

```
npm install
```

* **本地启动项目**

```
npm start
```

打开浏览器，进入`localhost:3000`访问前端页面。



## 本地部署步骤

> 支持 `windows` `linux` 系统。 

本地部署比较简单，我们将应用部署在局域网内，局域网内的相连通主机可以访问。

以下命令均在项目的根目录下进行。

* **项目打包，生成`build` 文件夹。**

```
npm run build 
```

* **安装 `serve` 包**。

```
npm install -g serve
```

* **命令运行开启服务。**

```
serve -s build
```

运行成功，我们会看到以下界面：

![](https://raw.githubusercontent.com/JoshuaQYH/blogImage/master/img/20190626232449.png)

第二个是我们的局域网`IP`，可以互`ping`联通的主机可以访问这个`IP`访问应用界面。（可能需要关闭防火墙~)

但这个办法由于没有提供公网 `IP` ，所以外网要进行访问，还是要将项目部署到云端，通过公网 `IP ` 来访问。

以下是云端部署应用的步骤介绍。



## 云端部署步骤

> 本部署教程支持 `linux` 系统。

这一部分将介绍如何利用`Docker`、 `Nginx` 将`React`项目部署到云服务器上。

* **通过以下命令将项目打包，生成`build`文件夹**

```
npm run build
```

接下来我们需要把把`build`文件夹部署到云服务器上，通过一个公网`IP`来访问我们的前端页面。

----

* **云服务器配置`Docker`环境** 

关于如何详细的选购服务器并在云端进行`React`的应用部署，可以看我的另外一篇博文——[使用 Docker、Nginx部署一个简单React Demo到云服务器上](<https://blog.csdn.net/CVSvsvsvsvs/article/details/93587223>)。以下命令内容均摘抄自此博文，方便快速部署。

说明一下，这里选择的腾讯云的服务器，系统是`CentOS7.6`。

我们首先需要配置`Docker`环境，安装过程参考自[官网CentOS教程](<https://docs.docker.com/install/linux/docker-ce/centos/>)。

1. **配置`Docker`存储库。**

```
// 安装必要的包
sudo yum install -y yum-utils \
		device-mapper-persistent-data \
  		lvm2
 
// 配置稳定版的docker仓库
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

2. **安装`Docker CE` 社区版**。

```
sudo yum install docker-ce docker-ce-cli containerd.io
```

3. **测试`Docker`是否安装成功。**

```
sudo systemctl start docker
sudo docker run hello-world
```

4. **安装容器管理工具 `Compose`**。

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

5. **测试`Compose`是否安装成功**。

```
docker-compose --version
```

----

* **云服务器部署应用**

1. **传输项目相关文件。**

在云服务器上建立一个文件夹用于存放`build`文件夹和`docker-compose.yml`,` nginx.conf`。

我们将项目中的`build`、`docker-compose.yml`、 `nginx.conf`都传输到服务器上。

> 如何传输？建议使用[WinSCP](<https://winscp.net/eng/docs/lang:chs>) 图形化工具进行传输，终端可使用`scp`命令传输。

建立的目录结构像这样：

```
App
  |__ build
  |__ docker-compose.yml
  |__ nginx.conf
```

2. **运行命令部署**。

在`App`文件夹的路径下，运行以下命令完成`Docker`容器的创建和`Nginx`的部署。

```
docker-compose up -d
```

创建容器成功后，我们就可以在通过`公网IP:80` 来访问应用啦。

比如你的服务器`IP`是`123.456.123.456` ，那么你就可以通过`123.456.123.456:80`访问挣钱宝应用啦。

如果你要关闭容器，那么你可以运行以下命令。

```
docker-compose down
```

到此位置项目就部署成功啦。

----

以下是本项目部署成功的截图~

![](https://raw.githubusercontent.com/JoshuaQYH/blogImage/master/img/20190626231246.png)


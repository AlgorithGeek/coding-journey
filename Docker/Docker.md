# 基本概念

> 首先要有一个认知，就是要先大概对Docker中的**“容器”**有点认识，这里的容器本质上是一个**“进程“**
>
> ”进程“最根本的本质是**资源容器**，所以这里的**“容器”**就是**进程**
>
> > 要是初学者的话，可以暂时这样理解：
> >
> > - **容器**是一个可以被运行的软件，这个软件的代码，就是我们放到容器中的代码，运行容器，就是运行我们的代码
> >
> > - 把代码及其和代码相关的各种东西都放到容器内部，从而进行统一，容器无论在哪里运行，因为内部统一，运行都不会有差异
>
> Docker实际上就是一个软件，是一个容器管理平台，只不过它的功能有点特殊而已

- **Docker的功能**：调用操作系统内核提供的能力，去**创建、管理和运行“容器”**



- **Docker**解决的问题：
  - **一致性问题**：通过打包应用和环境，解决了“我的电脑可以，你的不行”的这种运行差异问题
  - **部署效率问题**：通过标准化部署单元，解决了手动部署复杂、缓慢和高风险的问题
  - **依赖冲突问题**：通过隔离，解决了“依赖地狱”问题
  - **资源利用率问题**：通过轻量化，解决了虚拟机的资源浪费和高成本问题



# 虚拟机和Docker

- **虚拟机**：用来**模拟硬件**，之后我们下载的**操作系统**就可以在里面**操作这些模拟的“硬件”**
- **Docker**：用来**利用操作系统**，**创建、管理和运行“容器”**



# Docker安装

- 官方文档网址：https://docs.docker.com/engine/install/





# DockerFile

## **官方定义**

- A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build` users can create an automated build that executes several command-line instructions in succession.

  > Dockerfile 是一个文本文档，它包含了用户可以在命令行上调用以组装成一个镜像的所有命令。
  >
  > 通过使用 `docker build` 命令，用户可以创建一个自动化构建过程，该过程会连续执行多条命令行指令

- 唯一目的是**定义**如何一步步地构建一个 Docker 镜像。

  它详细描述了需要什么样的基础环境、安装哪些软件、拷贝什么文件进去，以及最后应该如何运行程序



# 镜像

## **官方定义**

- An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the `ubuntu` image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

  >镜像是一个带有创建 Docker 容器指令的**只读模板**。通常，一个镜像会基于另一个镜像，并加上一些额外的定制。
  >
  >例如，你可能会构建一个基于 `ubuntu` 镜像的镜像，但其中安装了 Apache Web 服务器和你的应用程序，以及运行你的应用所需的所有配置细节



# 容器

> 通过将**代码及其完整的运行环境**在**逻辑上放入** **容器**中，可以确保无论这个容器在哪里运行，其表现都完全一致，没有差异

## 官方定义

- A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. A container is defined by its image as well as any configuration options you provide to it at creation time. When a container is removed, any changes to its state that are not stored in persistent storage disappear.

  >容器是镜像的一个**可运行实例**。
  >
  >你可以使用 Docker API 或命令行工具（CLI）来创建、启动、停止、移动或删除容器。
  >
  >容器由其镜像以及在创建时提供给它的任何配置选项所定义。
  >
  >当一个容器被移除时，任何未存储在持久化存储中的状态变更都会消失

- **容器**是根据静态的**镜像**创建出来的。容器是一个**可以被启动、停止和删除**的实例，而镜像本身永远是静止不变的**模板**



# 仓库





# 数据卷
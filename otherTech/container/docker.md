# Docker
## 前置技术点
### Linux 中的 namespace
  在 Linux 中，namespace 是一种内核特性，用于隔离和限制进程之间的资源视图。每个命名空间提供了一组资源的独立实例，使得在不同命名空间中运行的进程看到的资源是不同的。

  Linux 提供了多种命名空间：

  1. **PID namespace:** 隔离进程 ID。在不同的 PID 命名空间中运行的进程拥有不同的进程 ID 空间，一个进程在其所属的 PID 命名空间中的进程 ID 在其他命名空间中看起来可能是另一个值。
  2. **Network namespace:** 隔离网络栈。每个网络命名空间都拥有自己的网络设备、IP地址、路由表和防火墙规则，使得在不同网络命名空间中运行的进程可以独立配置和管理网络。
  3. **Mount namespace:** 隔离文件系统挂载点。每个挂载命名空间拥有独立的文件系统挂载状态，使得在不同挂载命名空间中运行的进程可以拥有不同的文件系统视图。
  4. **UTS namespace:** 隔离主机名和域名。每个 UTS 命名空间拥有独立的主机名和域名，使得在不同 UTS 命名空间中运行的进程可以拥有不同的主机标识。
  5. **IPC namespace:** 隔离 System V IPC 和 POSIX 消息队列等进程间通信机制，每个 IPC 命名空间拥有独立的信号量、共享内存和消息队列，使得在不同 IPC 命名空间中运行的进程之间的通信互不干扰。
  6. **User namespace:** 隔离用户和用户组标识。每个用户命名空间拥有独立的用户和用户组标识，使得在不同用户命名空间中运行的进程可以拥有不同的用户身份。

  **通过使用命名空间， Linux 可以实现一种轻量级的虚拟化，允许在同一主机上同时运行多个相互隔离的进程集合，每个进程集合拥有自己独立的资源视图。这种隔离性使得命名空间在容器技术（如 Docker 和 Kubernetes）中得到广泛应用，用于实现容器之间的隔离和资源管理。**

### 容器镜像（rootfs）
  rootfs 是一个操作系统的所有文件和目录，并不不好喊内核。而相比之下，传统虚拟机的镜像大多是一个磁盘的“快照”,磁盘有多大，镜像就至少有多大。

  通过 mount namespace 和 rootfs 结合，容器就能够为进程构建出一个完整的文件系统隔离环境。这个功能主要依赖于 chroot 和 pivot_root 连个系统调用切换进程根目录的能力。

  通过 Cgroups 进行限制。

### 联合文件系统（Union File System）
  可以将多个不同位置的文件联合挂载。

### 容器特点：
  一个正在运行的 Linux 容器，分为两部分：

  - 容器镜像（containerImage）: 一组联合挂载在 ```/var/lib/docker/aufs/mnt``` 上的rootfs,是容器的静态视图。
  - 容器运行时（Container Runtime）：一个由 namespace + cgroups 构成的隔离环境，是容器的动态视图。

  1. 隔离和限制

    通过 Namespace 隔离

    通过 Cgroups 进行限制。

  2. 一致性：由于 rootfs（容器镜像） 的存在。

    由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封账在了一起。对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”。

    这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟。

    Docker在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。这里面使用到了一种叫作联合文件系统（Union File System）的能力。  
    
## 概念
  Docker 是一个容器. 是一个用于开发、发布和运行应用程序的平台。
### 什么是容器
  容器是代码的隔离环境。这意味着容器不知道您的操作系统或文件。

  它运行在 Docker Desktop 提供给您的环境上。

  容器拥有代码运行所需的一切，甚至是基本的操作系统。
### DockerInit
  Docker 创建的一个容器初始化进程，而不是应用进程（ENTRYPOINT + CMD）。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要再容器内进行的初始化操作。最后，它通过 ```execv()``` 系统调用，让应用进程取代自己，称为容器里的 PID =1 的进程。
### Docker Volume
  把一个宿主机的目录或者文件，挂载到容器中。
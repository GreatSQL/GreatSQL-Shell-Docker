# GreatSQL-Shell Docker

如何利用Docker环境快速编译MySQL Shell for GreatSQL二进制包，详情参考：[MySQL Shell 8.0.32 for GreatSQL编译安装](https://mp.weixin.qq.com/s/bB6ZvV6-yB83otLnv_oqrA)

相关文件介绍：
- Dockerfile，用于构建Docker编译环境
- greatsql-shell-build.sh，用于实现在Docker容器中自动化编译的脚本
- greatsql-shell-docker-build.sh，用于自动构建Docker编译环境的脚本
- mysqlsh-for-greatsql-8.0.32.patch，需要对MySQL Shell打补丁，才能支持GreatSQL中特有的仲裁节点特性

**提醒**：
1. 如果是在aarch64环境中，则修改Dockerfile的前几行，改成适用于aarch64的镜像即可，例如
```
#for x86_64
#FROM centos:8

#for aarch64
FROM docker.io/arm64v8/centos
```
2. 在Dockerfile中有个`COPY greatsql_shell_docker_build.tar /opt`的动作，需要自行打包 **greatsql_shell_docker_build.tar** 文件包，主要由一下几个文件组成：

- [antlr4-4.10.0.tar.gz](https://github.com/antlr/antlr4/archive/refs/tags/4.10.tar.gz)
- [boost_1_77_0.tar.gz](https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.gz)
- [mysql-8.0.32.tar.gz](https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.32.tar.gz)
- [mysql-shell-8.0.32-src.tar.gz](https://downloads.mysql.com/archives/get/p/43/file/mysql-shell-8.0.32-src.tar.gz)
- mysqlsh-for-greatsql-8.0.32.patch, 见本目录
- [patchelf-0.14.5.tar.gz](https://github.com/NixOS/patchelf/releases/download/0.14.5/patchelf-0.14.5.tar.gz)
- [protobuf-all-3.19.4.tar.gz](https://github.com/protocolbuffers/protobuf/releases/download/v3.19.4/protobuf-all-3.19.4.tar.gz)
- [rpcsvc-proto-1.4.tar.gz](https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4/rpcsvc-proto-1.4.tar.gz)

其中，几个在github上的文件需要科学上网才能下载。此外，在文档 **[MySQL Shell 8.0.32 for GreatSQL编译安装](https://mp.weixin.qq.com/s/bB6ZvV6-yB83otLnv_oqrA)** 中有提到，antlr4源码包中的`runtime/Cpp/runtime/CMakeLists.txt`文件第一行要注释掉并重新打包，否则编译时要去github上下载文件，会导致编译失败。

3. 上述 **greatsql_shell_docker_build.tar** 文件包准备好后，即可参考下面的方法完成自动化编译工作：

```
# 前面略过Docker的安装过程
# 直接拉取镜像并创建新容器
$ docker run -itd --hostname greatsqlsh --name greatsqlsh greatsql/greatsql_shell_build:8.0.32-25 bash

# 查看容器日志，大概要等几分钟才能编译完成，取决于服务器性能
# 如果看到类似下面的结果，就表明二进制包已编译完成
$ docker logs greatsqlsh | tail
1. extracting tarballs
2. compiling antlr4
3. compiling antlr4
4. compiling rpcsvc-proto
5. compiling protobuf
6. compiling greatsql shell
/opt/greatsql-shell-8.0.32-25-centos-glibc2.28-x86_64/bin/mysqlsh   Ver 8.0.32 for Linux on x86_64 - for MySQL 8.0.32 (Source distribution)
7. MySQL Shell 8.0.32-25 for GreatSQL build completed! TARBALL is:
-rw-r--r-- 1 root root 20343832 Jan 20 21:41 greatsql-shell-8.0.32-25-centos-glibc2.28-x86_64.tar.xz
```

接下来回退到宿主机，将容器中的二进制包拷贝出来
```
$ docker cp greatsqlsh:/opt/greatsql-shell-8.0.32-25-centos-glibc2.28-x86_64.tar.gz /usr/local/
```
然后解压缩，就可以在宿主机环境下使用了。
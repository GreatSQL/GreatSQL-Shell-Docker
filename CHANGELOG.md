# 更新日志

## 8.0.32-25(2024.1.22)

### 新功能

* 版本更新到GreatSQL 8.0.32-25。
* 支持从hub.docker.com中拉取镜像并自动完成编译工作。
* 也支持调用本地脚本完成自动创建GreatSQL Shell编译环境Docker镜像，并自动创建GreatSQL Shell自动编译Docker容器，一条命令即可完成全部编译工作。
* 编译后的二进制包用xz压缩，压缩比更高，在xz压缩时采用并行方式，降低压缩耗时。

[8.0.32-25]: https://gitee.com/GreatSQL/GreatSQL-Shell-Docker/tree/greatsql-8.0.32-25/

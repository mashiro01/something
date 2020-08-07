# docker 基本操作(三)：数据存储

docker中的数据修改是临时的，当我们退出容器后，我们所做的一切修改均会丢失。如果我们想要对数据进行永久性保存，这时我们就需要存储卷来进行数据的永久性保存

这里容器中管理数据的主要有两种方式

- 数据卷（Volumes）
- 挂载主机目录（Bind mounts）

## 数据卷

`数据卷`是可供一个或多个容器使用的特殊目录，它有以下特性

1. 数据卷可以在**容器间**`共享`和`重用`
2. 对数据卷的修改不会影响到镜像
3. 数据卷会一直存在，是一个独立的个体

这样不难看出，数据卷相当于一个独立的存储介质，每一个容器对数据卷的调用相当于`mount`该介质

### 创建数据卷

我们可以使用以下格式来创建一个数据卷

```shell
docker volume create <vol-name>
```

### 查看数据卷

```shell
docker volume ls
```

### 查看数据卷信息

```shell
docker volume inspect <vol-name>/<vol-ID>
```

### 使用数据卷

如果我们要使用创建好的数据卷，我们可以在启动容器时使用
`--mount source=<vol-name>,target=<mount-folder>`来进行数据卷的挂载

这里给出一个示例

```shell
docker run -it -P --name web_test --mount source=my-vol,target=/webapp me/ubuntu python app.py
```

### 删除数据卷

```shell
docker volume rm <vol-name>
```

> 如果我们需要在删除容器的时候同时删除数据卷，可以在后边加上`-v`这个选项

## 挂载主机目录

我们可以使用以下选项来挂载本机目录到容器中

```text
--mount type=bind,source=<local-folder>,target=<dst>
```

同时如果我们想要对挂载的权限进行限制，可以在后边加上`readonly`来指定挂载的目录为`只读状态`

### 文件的挂载

我们不仅能够挂载目录到容器中，同时能够挂载本机的文件到容器中来记录下容器的活动或者是直接使用主机的配置

这里我们只要把挂载的目录名替换为文件名即可

```text
--mount type=bind,source=<local-file>,target=<dst>
```

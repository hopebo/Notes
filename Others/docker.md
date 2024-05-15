# Docker

## 安装和配置

[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)

Docker daemon 线程会使用 root 权限绑定到 unix socket 上，只有 root 权限才能访问。通过将用户加入 docker 用户组可以在非 sudo 下也能访问 daemon 线程。[Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

```bash
sudo groupadd docker

# 可能需要安装 nscd
sudo usermod -aG docker $USER

newgrp docker
```

## 使用

### 文件映射

docker 可以将主机上的目录映射到容器中的指定路径，在主机和容器上的修改都是持久生效的。[Use bind mounts](https://docs.docker.com/get-started/06_bind_mounts/)

```bash
docker run -v /work/directory:/work -v $HOME:$HOME
```

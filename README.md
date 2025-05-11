![CF-Workers-docker.io](./img.png)

# 🐳 CF-Workers-docker.io：Docker仓库镜像代理工具

这个项目是一个基于 Cloudflare Workers 的 Docker 镜像代理工具。它能够中转对 Docker 官方镜像仓库的请求，解决一些访问限制和加速访问的问题。

> [!CAUTION]
> **docker.fxxk.dedyn.io 已被GFW污染，需自行部署使用。**

> [!WARNING]
> 根据 [Cloudflare 协议](https://www.cloudflare.com/zh-cn/terms/) 中，2.2.1 第 (j) use the Services to provide a virtual private network or other similar proxy services.
>
> 使用本服务可能存在被 Cloudflare 封号的潜在风险，请自行斟酌使用风险。

## 🚀 部署方式

- **Workers** 部署：复制 [_worker.js](https://github.com/cmliu/CF-Workers-docker.io/blob/main/_worker.js) 代码，`保存并部署`即可
- **Pages** 部署：`Fork` 后 `连接GitHub` 一键部署即可

## ⚙️ 如何使用？ [视频教程](https://www.youtube.com/watch?v=l2jwq9CagNQ)

例如您的Workers项目域名为：`docker.fxxk.dedyn.io`；

### 1.官方镜像路径前面加域名

```shell
docker pull docker.0037.fit/stilleshan/frpc:latest
```

```shell
docker pull docker.0037.fit/library/nginx:stable-alpine3.19-perl
```

### 2.一键设置镜像加速

修改文件 `/etc/docker/daemon.json`（如果不存在则创建）

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.0037.fit"]  # 请替换为您自己的Worker自定义域名
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 3. 配置常见仓库的镜像加速

#### 3.1 配置

`Containerd` 较简单，它支持任意 `registry` 的 `mirror`，只需要修改配置文件 `/etc/containerd/config.toml`，添加如下的配置：

```yaml
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://docker.0037.fit"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
          endpoint = ["https://docker.0037.fit"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
          endpoint = ["https://docker.0037.fit"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
          endpoint = ["https://docker.0037.fit"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
          endpoint = ["https://docker.0037.fit"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
          endpoint = ["https://docker.0037.fit"]
```

`Podman` 同样支持任意 `registry` 的 `mirror`，修改配置文件 `/etc/containers/registries.conf`，添加配置：

```yaml
unqualified-search-registries = ['docker.io', 'k8s.gcr.io', 'gcr.io', 'ghcr.io', 'quay.io']

[[registry]]
prefix = "docker.io"
insecure = true
location = "registry-1.docker.io"

[[registry.mirror]]
location = "docker.0037.fit"

[[registry]]
prefix = "registry.k8s.io"
insecure = true
location = "registry.k8s.io"

[[registry.mirror]]
location = "docker.0037.fit"

[[registry]]
prefix = "k8s.gcr.io"
insecure = true
location = "k8s.gcr.io"

[[registry.mirror]]
location = "docker.0037.fit"

[[registry]]
prefix = "gcr.io"
insecure = true
location = "gcr.io"

[[registry.mirror]]
location = "docker.0037.fit"

[[registry]]
prefix = "ghcr.io"
insecure = true
location = "ghcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "quay.io"
insecure = true
location = "quay.io"

[[registry.mirror]]
location = "docker.0037.fit"

```

#### 3.3 使用

对于以上配置，k8s 在使用的时候，就可以直接 `pull` 外部无法 pull 的镜像了。

```shell
# 手动可以直接pull配置了mirror的仓库
crictl pull registry.k8s.io/kube-proxy:v1.28.4
docker  pull nginx:1.21
```

## 🔧 变量说明

| 变量名 | 示例 | 必填 | 备注 |
|--|--|--|--|
| URL302 | `https://t.me/CMLiussss` |❌| 主页302跳转 |
| URL | `https://www.baidu.com/` |❌| 主页伪装(设为`nginx`则伪装为nginx默认页面) |
| UA | `netcraft` |❌| 支持多元素, 元素之间使用空格或换行作间隔 |

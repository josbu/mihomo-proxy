# Mihomo 一键安装脚本使用说明

## 简介

Mihomo 一键安装脚本（mihomo-docker.sh）是一个方便用户快速部署 Mihomo 代理服务的工具。通过简单的命令，即可自动完成 Docker 环境搭建、网络配置以及 Mihomo 容器的部署与启动。

## 目录结构

当前目录结构如下：

```
mihomo-docker/
├── mihomo-docker.sh          # 主安装脚本
├── README.md          # 本使用说明
└── files/             # 必要的配置文件和辅助脚本
    ├── config.yaml       # Mihomo配置模板
    ├── setup_proxy.sh    # 代理设置脚本
    ├── setup_router.sh   # 路由器配置脚本
    ├── check_status.sh   # 状态检查脚本
    └── mihomo_state.json # 状态配置文件
```

## 使用方法

### 1. 获取脚本

直接使用脚本所在目录的mihomo-docker.sh：

```bash
cd mihomo-docker
bash mihomo-docker.sh
```

如需下载到其他位置，可使用以下命令：

```bash
# 使用curl下载
curl -fsSL https://raw.githubusercontent.com/wallentv/mihomo-proxy/refs/heads/master/mihomo-docker/mihomo-docker.sh -o mihomo-docker.sh
```

```bash
# 或使用wget下载
wget -O mihomo-docker.sh https://raw.githubusercontent.com/wallentv/mihomo-proxy/refs/heads/master/mihomo-docker/mihomo-docker.sh
```

### 2. 运行脚本

```bash
chmod +x mihomo-docker.sh  # 赋予执行权限
bash mihomo-docker.sh      # 执行脚本
```

## 菜单选项说明

运行脚本后，您将看到以下菜单选项：

1. **分步安装** - 按步骤引导完成安装，可根据需要自定义每个步骤
2. **一键安装** - 自动完成所有安装步骤，无需手动干预
3. **检查状态** - 检查Mihomo安装和运行状态
4. **重启服务** - 重启Mihomo代理服务
5. **配置路由器** - 生成路由器配置命令
6. **卸载Mihomo** - 完全卸载Mihomo及其配置
0. **退出脚本** - 退出安装程序

## 安装方法详解

### 一键安装

选择菜单中的"一键安装"选项，脚本会自动完成以下步骤：

1. 检查系统环境
2. 初始化配置
3. 安装依赖和配置环境
4. 安装和配置Mihomo
5. 检查安装结果

整个过程无需手动干预，适合大多数用户使用。

### 分步安装（推荐）

选择菜单中的"分步安装"选项，脚本会引导您完成以下步骤：

1. **初始化设置** - 设置mihomo的IP地址并检查配置脚本
2. **配置代理机** - 安装Docker和Mihomo，配置网络
3. **配置路由器** - 生成路由器配置命令

分步安装适合希望自定义安装过程的高级用户。

## 安装完成后

安装完成后，您将看到类似以下的信息：

```
======================================================
            Mihomo 代理安装完成！
======================================================
控制面板地址: http://192.168.1.4:9090/ui
混合代理端口: 192.168.1.4:7890
HTTP代理端口: 192.168.1.4:7891
SOCKS代理端口: 192.168.1.4:7892
======================================================
```

您可以：

1. 通过浏览器访问控制面板：`http://[Mihomo_IP]:9090/ui`
2. 在客户端设备上配置代理，使用以下端口：
   - HTTP代理：`[Mihomo_IP]:7891`
   - SOCKS5代理：`[Mihomo_IP]:7892`
   - 混合代理：`[Mihomo_IP]:7890`
3. 使用"配置路由器"选项生成路由器配置命令，实现全局代理

## 常见问题解答

### 1. 如何检查 Mihomo 是否正常运行？

在脚本主菜单中选择"检查状态"选项，系统将显示当前服务运行状态。

### 2. 如何重启 Mihomo 服务？

在脚本主菜单中选择"重启服务"选项，系统将重启 Mihomo 容器。

### 3. 如何卸载 Mihomo？

在脚本主菜单中选择"卸载Mihomo"选项，按照提示完成卸载过程。

### 4. 如何更新 Mihomo？

重新运行安装脚本，然后选择"一键安装"选项即可更新到最新版本。

### 5. 如何修改配置文件？

Mihomo 的主配置文件位于 `/etc/mihomo/config.yaml`，您可以使用编辑器修改配置：

```bash
nano /etc/mihomo/config.yaml
```

修改完成后，使用脚本中的"重启服务"选项使配置生效。

## 特别说明

1. 脚本会自动加载files文件夹下的执行脚本和配置文件
2. 如果您没有下载完整的目录结构，脚本会尝试从当前目录查找必要文件
3. 如果您有自定义的配置文件，请将其放置在同一目录下或创建files目录
4. 安装过程中如遇问题，可选择"检查状态"选项查看详细信息
5. 更新Mihomo只需重新运行脚本并选择"一键安装"选项

## 系统要求

- 支持系统：Debian 10/11/12, Ubuntu 20.04/22.04/24.04
- 最低配置：1核CPU、1GB内存、5GB存储空间
- 网络要求：可正常访问互联网 
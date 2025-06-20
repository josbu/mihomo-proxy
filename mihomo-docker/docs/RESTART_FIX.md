# Mihomo 重启服务修复说明

## 问题描述

之前的重启服务功能存在一个严重问题：**重启服务时会重置用户的配置文件**。

### 原问题流程

1. 用户选择"重启服务"
2. 脚本调用 `bash setup_proxy.sh restart`
3. `setup_proxy.sh` 执行完整安装流程
4. **用户自定义的 `config.yaml` 被模板文件覆盖**
5. 用户丢失所有自定义配置（代理服务器、规则等）

## 修复方案

### 1. 修改重启服务逻辑

**修复前**：
```bash
# 调用完整安装脚本
bash "$PROXY_SCRIPT" restart
```

**修复后**：
```bash
# 直接重启容器，保留配置文件
docker stop mihomo
docker rm mihomo
docker run -d --name=mihomo ... -v /etc/mihomo:/root/.config/mihomo ...
```

### 2. 智能配置文件处理

**修复前**：
```bash
# 总是覆盖配置文件
cp "$config_template" "$CONF_DIR/config.yaml"
```

**修复后**：
```bash
# 检查配置文件是否已存在
if [[ -f "$CONF_DIR/config.yaml" ]]; then
    echo "配置文件已存在，跳过复制，保留现有配置"
else
    cp "$config_template" "$CONF_DIR/config.yaml"
fi
```

### 3. 支持重启模式

`setup_proxy.sh` 现在支持两种模式：

- **安装模式**（默认）：`bash setup_proxy.sh`
- **重启模式**：`bash setup_proxy.sh restart`

## 修复后的行为

### 重启服务时

1. ✅ **保留用户配置**：不会覆盖 `/etc/mihomo/config.yaml`
2. ✅ **智能网络选择**：优先尝试macvlan，失败时自动降级到bridge
3. ✅ **状态检测**：检查容器是否正常启动
4. ✅ **访问信息**：根据网络类型显示正确的访问地址

### 完整安装时

1. ✅ **智能配置**：如果配置文件已存在，不会覆盖
2. ✅ **首次安装**：如果配置文件不存在，使用模板创建
3. ✅ **向下兼容**：保持原有的安装流程

## 使用示例

### 场景1：用户已自定义配置，需要重启服务

```bash
# 用户修改了 /etc/mihomo/config.yaml，添加了自己的代理服务器
nano /etc/mihomo/config.yaml

# 重启服务（配置文件会被保留）
sudo bash mihomo.sh
# 选择 "4. 重启服务"
```

**结果**：
- ✅ 容器重启成功
- ✅ 用户配置完全保留
- ✅ 代理服务正常工作

### 场景2：首次安装

```bash
# 首次运行安装脚本
sudo bash mihomo.sh
# 选择 "2. 一键安装"
```

**结果**：
- ✅ 创建默认配置文件
- ✅ 安装并启动服务
- ✅ 用户可以后续修改配置

### 场景3：重新安装但保留配置

```bash
# 用户已有自定义配置，但需要重新安装
sudo bash mihomo.sh
# 选择 "2. 一键安装"
```

**结果**：
- ✅ 重新安装Docker和网络
- ✅ 保留现有配置文件
- ✅ 启动服务使用用户配置

## 配置文件保护机制

### 1. 检查机制

```bash
if [[ -f "/etc/mihomo/config.yaml" ]]; then
    echo "配置文件已存在，保留现有配置"
else
    echo "创建新的配置文件"
fi
```

### 2. 备份建议

虽然脚本现在会保护配置文件，但仍建议用户定期备份：

```bash
# 备份配置文件
cp /etc/mihomo/config.yaml /etc/mihomo/config.yaml.backup.$(date +%Y%m%d)

# 查看备份
ls -la /etc/mihomo/config.yaml.backup.*
```

### 3. 恢复配置

如果需要恢复到默认配置：

```bash
# 删除现有配置
sudo rm /etc/mihomo/config.yaml

# 重新运行安装脚本
sudo bash mihomo.sh
# 选择 "2. 一键安装"
```

## 验证修复

### 测试步骤

1. **安装Mihomo**：
   ```bash
   sudo bash mihomo.sh  # 选择一键安装
   ```

2. **修改配置文件**：
   ```bash
   sudo nano /etc/mihomo/config.yaml
   # 添加一些自定义内容，如注释
   ```

3. **重启服务**：
   ```bash
   sudo bash mihomo.sh  # 选择重启服务
   ```

4. **验证配置保留**：
   ```bash
   sudo cat /etc/mihomo/config.yaml
   # 检查自定义内容是否还在
   ```

### 预期结果

- ✅ 配置文件中的自定义内容应该完全保留
- ✅ 容器应该成功重启
- ✅ 代理服务应该正常工作
- ✅ 控制面板应该可以访问

## 注意事项

1. **配置文件位置**：`/etc/mihomo/config.yaml`
2. **权限要求**：需要root权限修改配置文件
3. **配置验证**：修改配置后建议验证YAML语法
4. **服务重启**：配置修改后需要重启服务才能生效

## 新增功能

### 1. 彻底卸载功能

修复后的卸载功能现在会彻底清理所有资源：

- ✅ **删除Docker容器和网络**：包括mnet macvlan网络
- ✅ **删除主机网络接口**：清理macvlan接口
- ✅ **删除混杂模式服务**：清理systemd服务
- ✅ **删除配置文件**：清理所有配置和状态文件
- ✅ **清理网络路由**：删除可能的静态路由规则

### 2. 重置配置文件功能

新增的重置配置功能允许用户：

- 🔄 **重置配置文件**：恢复为默认模板配置
- 💾 **自动备份**：重置前自动备份当前配置
- 🚀 **自动重启**：重置后自动重启服务
- 📊 **状态显示**：显示配置文件信息和重置结果

#### 使用场景

**场景1：配置文件损坏**
```bash
# 配置文件格式错误导致服务无法启动
sudo bash mihomo.sh
# 选择 "6. 重置配置"
```

**场景2：想要重新开始配置**
```bash
# 想要清除所有自定义配置，重新开始
sudo bash mihomo.sh
# 选择 "6. 重置配置"
```

**场景3：恢复默认设置**
```bash
# 测试完成后想要恢复到默认状态
sudo bash mihomo.sh
# 选择 "6. 重置配置"
```

## 菜单更新

主菜单现在包含7个选项：

```
[1] 1. 分步安装      - 按步骤引导完成安装
[2] 2. 一键安装      - 自动完成所有安装步骤
[3] 3. 检查状态      - 检查Mihomo安装和运行状态
[4] 4. 重启服务      - 重启Mihomo代理服务（保留配置）
[5] 5. 配置路由器    - 生成路由器配置命令
[6] 6. 重置配置      - 重置配置文件并重启服务
[7] 7. 卸载Mihomo    - 完全卸载Mihomo及其配置
[0] 0. 退出脚本
```

## 总结

修复后的系统提供：

- 🔒 **配置安全**：用户配置永远不会被意外覆盖
- 🔄 **智能重启**：自动选择最佳网络配置
- 🧹 **彻底清理**：完全卸载所有相关资源
- 🔄 **配置重置**：安全地重置配置文件
- 📝 **清晰提示**：明确告知用户每个操作的影响
- 🛡️ **向下兼容**：不影响现有的安装流程

现在用户可以：
- 安全地重启Mihomo服务而不丢失配置
- 彻底卸载Mihomo而不留下残留文件
- 方便地重置配置文件到默认状态 
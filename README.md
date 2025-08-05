# Clash Linux 服务器配置

[![GitHub](https://img.shields.io/badge/GitHub-xiaoxishui123%2Fclash-blue?style=flat-square&logo=github)](https://github.com/xiaoxishui123/clash)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

一个完整的 Clash 科学上网配置方案，适用于 Linux 服务器环境。包含详细的安装配置指南、常见问题排查和服务管理命令。

## 📋 目录

- [特性](#特性)
- [快速开始](#快速开始)
- [详细配置](#详细配置)
- [服务管理](#服务管理)
- [常见问题](#常见问题)
- [贡献](#贡献)
- [许可证](#许可证)

## ✨ 特性

- 🚀 **一键部署**：完整的 Linux 服务器 Clash 配置流程
- 📚 **详细文档**：包含安装、配置、维护的完整指南
- 🔧 **问题排查**：常见问题的解决方案和排查步骤
- 🛠️ **服务管理**：便捷的启动、停止、重启命令
- 🔒 **安全配置**：合理的文件权限和网络设置
- 📱 **远程管理**：支持远程可视化管理界面

## 🚀 快速开始

### 环境要求

- Linux 服务器（推荐 Ubuntu 18.04+ 或 CentOS 7+）
- 管理员权限（root 或 sudo）
- 网络连接正常

### 一键安装

```bash
# 1. 克隆仓库
git clone https://github.com/xiaoxishui123/clash.git
cd clash

# 2. 下载 Clash 核心程序
wget https://downloads.clash.wiki/ClashPremium/clash-linux-amd64-2023.08.17.gz
gunzip clash-linux-amd64-2023.08.17.gz
mv clash-linux-amd64-2023.08.17 clash
chmod +x clash

# 3. 启动服务
nohup ./clash -d . > clash.log 2>&1 &

# 4. 检查服务状态
ps aux | grep clash | grep -v grep
```

### 验证安装

```bash
# 设置代理
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 测试连接
curl cip.cc
```

## 📖 详细配置

### 1. 配置文件准备

#### 方法一：使用 Clash Verge 导出
1. 在 Windows 上使用 Clash Verge 导出 YAML 配置文件
2. 上传到服务器并重命名为 `config.yaml`

#### 方法二：使用订阅链接
```bash
wget -O config.yaml "你的订阅链接"
base64 -d config.yaml > config_decoded.yaml
mv config_decoded.yaml config.yaml
```

### 2. 关键配置参数

```yaml
# 全局代理模式
mode: global

# 控制端口（避免冲突）
external-controller: '127.0.0.1:5000'

# 混合端口
mixed-port: 7890
```

### 3. 节点管理

```bash
# 查看所有代理组和节点
curl -X GET "http://127.0.0.1:5000/proxies"

# 切换代理组
curl -X PUT "http://127.0.0.1:5000/proxies/EEVPN" \
  -H "Content-Type: application/json" \
  -d '{"name":"香港 02D7"}'
```

## 🛠️ 服务管理

### 基本命令

| 操作 | 命令 |
|------|------|
| 启动服务 | `nohup ./clash -d . > clash.log 2>&1 &` |
| 停止服务 | `pkill clash` |
| 重启服务 | `pkill clash && nohup ./clash -d . > clash.log 2>&1 &` |
| 查看状态 | `ps aux \| grep clash \| grep -v grep` |
| 查看日志 | `tail -f clash.log` |

### 目录迁移

如果从其他目录迁移配置：

```bash
# 1. 停止服务
pkill clash

# 2. 复制文件
cp /path/to/old/config.yaml /home/clash/
cp /path/to/old/Country.mmdb /home/clash/
cp /path/to/old/clash /home/clash/clash

# 3. 设置权限
chmod +x clash

# 4. 重启服务
nohup ./clash -d . > clash.log 2>&1 &
```

## 🔧 常见问题

### 权限问题

**问题**：`Permission denied` 错误
```bash
# 解决方案
chmod +x clash
```

### 文件问题

**问题**：找不到 clash 可执行文件
```bash
# 检查文件
ls -la clash*

# 重命名（如果需要）
mv clash.bak clash
```

### 端口问题

**问题**：端口冲突
```bash
# 检查端口占用
lsof -i:7890
lsof -i:5000

# 杀死占用进程
kill -9 <PID>
```

### 网络问题

**问题**：无法连接外网
- 检查防火墙设置
- 确认 7890 端口开放
- 验证 DNS 配置

## 📁 文件结构

```
clash/
├── clash                 # Clash 核心程序
├── config.yaml          # 配置文件
├── Country.mmdb         # GeoIP 数据库
├── clash_linux_setup.md # 详细配置文档
├── .gitignore          # Git 忽略文件
└── README.md           # 项目说明文档
```

## 🔒 安全注意事项

1. **配置文件安全**：`config.yaml` 包含敏感信息，请妥善保管
2. **访问控制**：建议将仓库设置为私有
3. **端口安全**：确保控制端口（5000）不被外部访问
4. **日志管理**：定期清理日志文件

## 🌐 远程管理

### SSH 端口转发

```bash
# 本地执行
ssh -L 5000:127.0.0.1:5000 root@你的服务器IP

# 浏览器访问
http://127.0.0.1:5000
```

### 推荐 Dashboard

- [yacd](https://github.com/haishanh/yacd) - 美观的 Clash Dashboard
- [Clash Dashboard](https://github.com/Dreamacro/clash-dashboard) - 官方 Dashboard

## 📝 配置文件备份

```bash
# 备份配置
cp config.yaml config.yaml.backup.$(date +%Y%m%d)

# 恢复配置
cp config.yaml.backup.20231201 config.yaml
```

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

- [Clash](https://github.com/Dreamacro/clash) - 优秀的代理工具
- [Clash Premium](https://github.com/Dreamacro/clash/releases) - 增强版 Clash
- [yacd](https://github.com/haishanh/yacd) - 美观的 Dashboard

---

## 📋 更新记录

### 2025年1月14日 - 端口配置优化
- **重要变更**：控制端口从 9919 改为 5000
- **变更原因**：解决与 dify-on-wechat 项目的端口冲突
- **影响范围**：
  - API 访问地址变更：`http://127.0.0.1:5000`
  - SSH 隧道命令变更：`ssh -L 5000:127.0.0.1:5000`
  - 端口检查命令变更：`lsof -i:5000`
- **配置文件**：`external-controller: '127.0.0.1:5000'`

### 端口分配情况
- **5000**：Clash Dashboard 控制端口（当前使用）
- **7890**：Clash 代理端口（TCP/UDP）
- **9919**：预留给 dify-on-wechat 项目
- **9001**：预留给 CapCutAPI 服务

---

⭐ 如果这个项目对你有帮助，请给它一个星标！
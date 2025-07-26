# Linux服务器 Clash 科学上网配置流程

## 1. 下载 Clash Premium 核心程序

```bash
cd /home
wget https://downloads.clash.wiki/ClashPremium/clash-linux-amd64-2023.08.17.gz
gunzip clash-linux-amd64-2023.08.17.gz
mv clash-linux-amd64-2023.08.17 clash
chmod +x clash
```

---

## 2. 创建配置目录并准备文件

```bash
# 创建 clash 配置目录
mkdir -p /home/clash
cd /home/clash

# 将 clash 程序移动到配置目录
mv /home/clash /home/clash/clash
```

---

## 3. 获取并准备 Clash 配置文件

### 方法一：用 Clash Verge 导出配置文件
- 在 Windows 上用 Clash Verge 导出 YAML 配置文件（如 `RjKoa13hb1Vm.yaml`）。
- 上传到服务器 `/home/clash/` 目录，并重命名为 `config.yaml`。

### 方法二：用订阅链接生成配置文件
- 直接用 curl/wget 下载订阅内容并 base64 解码（如机场提供的订阅链接）：
  ```bash
  wget -O config.yaml "你的订阅链接"
  base64 -d config.yaml > config_decoded.yaml
  mv config_decoded.yaml config.yaml
  ```

---

## 4. 上传 geoip 数据库（Country.mmdb）

- 下载 [Country.mmdb](https://github.com/Dreamacro/maxmind-geoip/raw/release/Country.mmdb) 到本地，再上传到 `/home/clash/` 目录。

---

## 5. 修改配置文件关键参数

- 推荐设置为全局代理：
  ```yaml
  mode: global
  ```
- 修改 Dashboard 控制端口，避免冲突（如 9919）：
  ```yaml
  external-controller: '127.0.0.1:9919'
  ```

---

## 6. 启动 Clash 服务

```bash
cd /home/clash
nohup ./clash -d . > clash.log 2>&1 &
```

---

## 7. 检查服务状态

- 查看进程是否运行：
  ```bash
  ps aux | grep clash | grep -v grep
  ```

- 查看日志确认端口监听和节点状态：
  ```bash
  tail -n 30 clash.log
  ```
  日志应包含：
  ```
  inbound create success inbound=mixed addr=:7890 network=tcp
  inbound create success inbound=mixed addr=:7890 network=udp
  [API] listening addr=127.0.0.1:9919
  ```

---

## 8. 设置系统代理并测试

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=localhost,127.0.0.1
curl cip.cc
```
- 若显示为境外 IP，说明代理生效。

---

## 9. 手动切换节点（命令行方式）

1. 查看所有代理组和节点：
   ```bash
   curl -X GET "http://127.0.0.1:9919/proxies"
   ```
2. 切换 GLOBAL 组到 EEVPN：
   ```bash
   curl -X PUT "http://127.0.0.1:9919/proxies/GLOBAL" -H "Content-Type: application/json" -d '{"name":"EEVPN"}'
   ```
3. 切换 EEVPN 组到指定节点（如"香港 02D7"）：
   ```bash
   curl -X PUT "http://127.0.0.1:9919/proxies/EEVPN" -H "Content-Type: application/json" -d '{"name":"香港 02D7"}'
   ```

---

## 10. 远程可视化管理（可选）

- 用 SSH 端口转发访问 Dashboard：
  ```bash
  ssh -L 9919:127.0.0.1:9919 root@你的服务器IP
  ```
- 本地浏览器访问：
  ```
  http://127.0.0.1:9919
  ```
- 如需更美观的前端，可用 [yacd](https://github.com/haishanh/yacd) 等开源 Dashboard。

---

## 11. 目录迁移和配置修复

### 从其他目录迁移到 /home/clash
如果之前在其他目录（如 /home/dify）配置过 clash，迁移步骤：

1. 停止原 clash 服务：
   ```bash
   pkill clash
   ```

2. 复制配置文件到新目录：
   ```bash
   cp /home/dify/config.yaml /home/clash/
   cp /home/dify/Country.mmdb /home/clash/
   cp /home/dify/clash /home/clash/clash
   ```

3. 确保文件权限正确：
   ```bash
   cd /home/clash
   chmod +x clash
   ls -la clash*
   ```

4. 重新启动服务：
   ```bash
   cd /home/clash
   nohup ./clash -d . > clash.log 2>&1 &
   ```

---

## 12. 常见问题与排查

### 权限问题
- **Permission denied 错误**：
  ```bash
  chmod +x clash
  ```

### 文件问题
- **找不到 clash 可执行文件**：
  ```bash
  ls -la clash*
  # 如果文件名为 clash.bak，需要重命名
  mv clash.bak clash
  ```

### 端口问题
- **7890 端口冲突**：用 `lsof -i:7890` 查找并杀死占用进程，确保 Clash 独占监听。
- **Dashboard 端口冲突**：修改 `external-controller` 端口，避免与其他服务冲突。

### 服务问题
- **节点切换无效**：确认节点 alive 状态，尝试切换其他节点。
- **curl 无响应**：检查 clash.log 日志，确认端口监听和节点健康。
- **服务启动失败**：检查配置文件语法，确保 YAML 格式正确。

### 网络问题
- **无法连接外网**：检查防火墙设置，确保 7890 端口开放。
- **DNS 解析失败**：检查 config.yaml 中的 DNS 配置。

---

## 13. 服务管理命令

### 启动服务
```bash
cd /home/clash
nohup ./clash -d . > clash.log 2>&1 &
```

### 停止服务
```bash
pkill clash
```

### 重启服务
```bash
pkill clash
cd /home/clash
nohup ./clash -d . > clash.log 2>&1 &
```

### 查看服务状态
```bash
ps aux | grep clash | grep -v grep
```

### 查看实时日志
```bash
cd /home/clash
tail -f clash.log
```

---

## 14. 配置文件备份和恢复

### 备份配置
```bash
cd /home/clash
cp config.yaml config.yaml.backup.$(date +%Y%m%d)
```

### 恢复配置
```bash
cd /home/clash
cp config.yaml.backup.20231201 config.yaml
```

---

如需进一步自动化脚本、节点健康监控、或遇到特殊问题，欢迎随时补充！ 
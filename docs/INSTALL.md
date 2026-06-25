# 安装与部署指南

## 首次运行与管理员密码

SimAdmin 采用后台式单管理员密码登录，不包含用户名和多账号权限系统。首次打开 Web 后台时会进入 `/login` 的“设置管理员密码”页面；设置成功后会自动建立会话并进入管理后台。

### 密码规则

- 8-64 个字符。
- 只能使用英文字母、数字和符号，不允许空格或中文。
- 至少包含两类字符，例如字母 + 数字、字母 + 符号或数字 + 符号。

### 登录与接口保护

- 管理后台页面和 `/api/*` 业务接口默认需要登录；`/api/health`、`/api/auth/status`、`/api/auth/setup`、`/api/auth/login` 为公开接口。
- 未登录访问受保护页面会跳转到 `/login`；前端 API 请求遇到 `401` 会自动进入登录页，直接调用 API 时返回标准 JSON 错误。
- 会话使用 `simadmin_session` HttpOnly Cookie，默认有效期 7 天。重置或清除管理员密码会清空所有 Web 会话。
- 当前不提供手动登出入口，适合单管理员设备后台场景。

### 忘记/清空密码

忘记密码时，可通过 SSH 登录目标设备后执行交互式重置：

```bash
/opt/simadmin/simadmin auth reset-password
```

如需清除管理员密码并让 Web UI 下次重新进入首次设置：

```bash
/opt/simadmin/simadmin auth clear
```

如果使用了自定义安装目录，请将 `/opt/simadmin/simadmin` 替换为实际后端二进制路径。

---

## 设备侧一键安装 / 升级

在目标设备上以 root 执行：

```bash
curl -fsSL https://raw.githubusercontent.com/3899/SimAdmin/main/install_latest.sh | sh
```

### 国内网络环境

```bash
curl -fsSL https://gh-proxy.com/https://raw.githubusercontent.com/3899/SimAdmin/main/install_latest.sh | sh
```

### 可选环境变量

```bash
curl -fsSL https://raw.githubusercontent.com/3899/SimAdmin/main/install_latest.sh \
  | REPO=3899/SimAdmin INSTALL_DIR=/opt/simadmin SERVICE_NAME=simadmin sh
```

### 安装脚本动作说明

- 从 GitHub Release 下载 `simadmin.tar.gz`。
- 安装后端二进制到 `/opt/simadmin/simadmin`。
- 安装前端到 `/opt/simadmin/www`。
- 安装并启用 `simadmin.service`。
- 安装并启用 `simadmin-modem-recovery.service`。
- 配置 NetworkManager 忽略 `wwan*` 接口，避免与 SimAdmin 抢占蜂窝连接管理。

---

## 设备侧一键卸载

默认彻底卸载，删除服务、程序文件、前端文件、OTA 临时目录、NetworkManager 配置以及用户数据：

```bash
curl -fsSL https://raw.githubusercontent.com/3899/SimAdmin/main/uninstall.sh | sh
```

### 国内网络环境

```bash
curl -fsSL https://gh-proxy.com/https://raw.githubusercontent.com/3899/SimAdmin/main/uninstall.sh | sh
```

### 保留用户数据卸载

如需保留短信数据库和配置文件：

```bash
curl -fsSL https://raw.githubusercontent.com/3899/SimAdmin/main/uninstall.sh \
  | sh -s -- --keep-user-data
```

### 自定义环境卸载

自定义安装路径或服务名时，需要和安装时保持一致：

```bash
curl -fsSL https://raw.githubusercontent.com/3899/SimAdmin/main/uninstall.sh \
  | INSTALL_DIR=/opt/simadmin SERVICE_NAME=simadmin sh -s -- --keep-user-data
```

### 卸载脚本参数说明

| 参数 | 说明 |
|------|------|
| `--purge` | 删除全部 SimAdmin 文件和用户数据，默认行为 |
| `--keep-user-data` | 保留 `/opt/simadmin/data.db`、SQLite sidecar 文件和配置文件 |
| `--install-dir PATH` | 指定安装目录，默认 `/opt/simadmin` |
| `--service-name NAME` | 指定主服务名，默认 `simadmin` |

### 卸载脚本动作说明

- 停止并禁用 `simadmin.service`。
- 停止并禁用 `simadmin-modem-recovery.service`。
- 删除 systemd 单元文件并执行 `daemon-reload` / `reset-failed`。
- 删除 `/usr/local/bin/simadmin-modem-recovery.sh`。
- 删除 `/etc/NetworkManager/conf.d/99-simadmin-unmanaged-modem.conf`，并在 NetworkManager 运行时重启它。
- 删除 `/tmp/ota_staging`。
- 默认删除 `/opt/simadmin` 和 `/data/config.json`；使用 `--keep-user-data` 时保留用户数据。

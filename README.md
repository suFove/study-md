# macOS 从零配置 OpenClaw 完整指南  
（网络修复 / Node / npm / Git SSH / Homebrew / 飞书插件 / 常用运维）

> 适用对象：在 macOS 上安装并运行 OpenClaw（含 gateway / 插件），并需要完整开发环境的人。  
> ⚠️ 注意：涉及密钥（appSecret / SSH key）的地方请使用占位符，不要泄露真实信息。

---

# 目录

1. 网络异常修复（macOS）
2. Apple 更新服务异常修复（可选）
3. 双系统恢复 macOS（Boot Camp 清理）
4. 安装 Node.js + npm 镜像
5. 解决 npm 全局权限问题（推荐）
6. Git / Xcode CLI 安装
7. GitHub SSH 配置（解决 publickey 报错）
8. 安装 Homebrew
9. 安装 OpenClaw
10. 飞书插件完整配置
11. 常用网站清单
12. OpenClaw 常用命令
13. 重启前后操作清单
14. 最小自检清单

---

# 1️⃣ macOS 网络异常修复

当出现：

- Wi-Fi 连不上
- 网络异常
- 系统代理错乱
- 虚拟网卡冲突

可尝试清理网络缓存。

## 步骤

进入目录：

```
/Library/Preferences/SystemConfiguration/
```

删除以下文件：

```
com.apple.airport.preferences.plist
com.apple.network.identification.plist
com.apple.wifi.message-tracer.plist
NetworkInterfaces.plist
preferences.plist
```

然后：

```
关机 → 再开机
```

⚠️ 注意：
- 会丢失 Wi-Fi 记录
- 若开启 VPN，先关闭系统代理

---

# 2️⃣ Apple 服务无法连接（旧系统可选）

若恢复或更新时报：

```
无法与服务器取得联系
```

可尝试设置 Software Update Catalog。

## 情况 A

```bash
sudo nvram lASUCatalogURL=https://swscan.apple.com/content/catalogs/others/index-10.16seed10.16-10.15-10.14-10.13-10.12-10.11-10.10-10.9-mountainlion-lion-snowleopard.leopard.merged-1.sucatalog
```

## 情况 B

```bash
sudo nvram IASUCatalogURL=https://swscan.apple.com/content/catalogs/others/index-10.16seed-10.16-10.15-10.14-10.13-10.12-10.11-10.10-10.9-mountainlion-lion-snowleopard-leopard.merged-1.sucatalog
```

⚠️ 修改的是 NVRAM，谨慎使用。

---

# 3️⃣ 双系统清理（Boot Camp）

操作顺序：

1. Boot Camp 删除 Windows
2. 磁盘工具抹除 Windows 分区
3. 若恢复报错 → 参考第 2 节

⚠️ 抹除磁盘前务必备份

---

# 4️⃣ 安装 Node.js（示例 Node 22）

## 安装

下载 dmg 安装包后双击安装。

## 验证

```bash
node -v
npm -v
```

---

# 5️⃣ 配置 npm 国内镜像

## 方式 1（直接设置）

```bash
npm config set registry https://registry.npmmirror.com/
```

## 方式 2（使用 nrm）

```bash
npm install -g nrm
nrm ls
nrm use taobao
nrm test
```

---

# 6️⃣ 解决 npm 全局权限问题（强烈推荐）

避免频繁 sudo。

## 1 创建目录

```bash
mkdir -p ~/.npm-global
```

## 2 设置 npm prefix

```bash
npm config set prefix '~/.npm-global'
```

## 3 加入 PATH（zsh）

```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## 验证

```bash
echo $PATH
which npm
```

---

# 7️⃣ 安装 Xcode Command Line Tools

```bash
xcode-select --install
```

---

# 8️⃣ GitHub SSH 修复（Permission denied publickey）

## 1 生成 SSH key

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

生成：

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

## 2 添加到 GitHub

```bash
cat ~/.ssh/id_rsa.pub
```

GitHub → Settings → SSH and GPG keys → New SSH key

## 3 测试连接

```bash
ssh -T git@github.com
```

成功示例：

```
Hi username! You've successfully authenticated.
```

## 4 确认远程地址使用 SSH

```bash
git remote -v
git remote set-url origin git@github.com:username/repository.git
```

---

# 9️⃣ 安装 Homebrew openclaw安装skills需要

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/usr/local/bin/brew shellenv)"
```

验证：

```bash
brew -v
```

---

# 🔟 安装 OpenClaw

## 安装

```bash
npm install -g openclaw@latest
```

## 初始化

```bash
openclaw onboard --install-daemon
```

## 配置

```bash
openclaw configure
```

---

# 1️⃣1️⃣ 飞书插件完整配置

## 安装插件

```bash
openclaw plugins install @m1heng-clawd/feishu
```

---

## 配置 Scopes 示例

```json
{
  "scopes": {
    "tenant": [
      "bitable:app",
      "bitable:app:readonly",
      "contact:contact.base:readonly",
      "contact:user.base:readonly",
      "docx:document",
      "docx:document.block:convert",
      "docx:document:readonly",
      "drive:drive",
      "drive:drive:readonly",
      "im:chat:readonly",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.group_msg",
      "im:message.p2p_msg:readonly",
      "im:message.reactions:read",
      "im:message:readonly",
      "im:message:recall",
      "im:message:send_as_bot",
      "im:message:update",
      "im:resource",
      "wiki:wiki",
      "wiki:wiki:readonly"
    ],
    "user": []
  }
}
```

---

## 设置 appId / appSecret

⚠️ 使用占位符

```bash
openclaw config set channels.feishu.appId "cli_xxxxxxxxxxxxxxxxx"
openclaw config set channels.feishu.appSecret "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
openclaw config set channels.feishu.enabled true
openclaw config set channels.feishu.connectionMode "websocket"
```

---

# 1️⃣2️⃣ 常用网站清单

- 邮箱：https://mail.163.com/
- 飞书开发平台：https://open.feishu.cn/app
- MiniMax：https://www.minimax.io/
- Brave Search API：https://brave.com/search/api/

---

# 1️⃣3️⃣ OpenClaw 常用命令

## 安装 / 初始化

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
openclaw configure
```

## 网关管理

```bash
openclaw gateway start
openclaw gateway restart
openclaw gateway stop
```

---

# 1️⃣4️⃣ 重启操作清单

## 关机前

```bash
openclaw gateway stop
```

- 关闭 VPN 系统代理
- 关闭虚拟网卡

## 重启后

```bash
source ~/.zshrc
openclaw gateway start
```

---

# ✅ 最小自检清单

```bash
node -v
npm -v
npm config get registry
which openclaw
ssh -T git@github.com
```

确认：

- openclaw 可执行
- npm 镜像正确
- SSH 正常
- 飞书 appId/appSecret 已配置
- connectionMode = websocket

---

# 🎯 至此完成

你已经拥有：

- 干净网络环境
- Node + npm 优化
- Git SSH 正常
- Homebrew
- OpenClaw daemon
- 飞书 WebSocket 机器人


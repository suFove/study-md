
macOS 上从零配置 OpenClaw：网络修复、Node/npm、Git、Homebrew、飞书插件与常用运维命令（含踩坑记录）

适用对象：在 macOS 上安装并运行 OpenClaw（含网关/插件），并需要配置 Node/npm、Git SSH、Homebrew、飞书插件等配套环境的人。
说明：本文包含一些“踩坑修复命令”，请按需执行；涉及账号密钥的内容务必使用占位符，不要把真实密钥写进博客或提交到仓库。


0. 准备工作与依赖概览

你最终会完成这些事：
- 修复 macOS 可能出现的网络异常/系统服务不可用问题
- 安装 Node.js（示例：Node 22.x）与 npm 国内镜像（含 nrm）
- 配置 Git + GitHub SSH，避免 Permission denied (publickey)
- 安装 Homebrew
- 安装并配置 OpenClaw（含网关 daemon）
- 安装并配置飞书插件（WebSocket 模式）
- 汇总“重启前/后”的操作清单，降低故障概率


1. macOS 连不上网：清理网络相关 plist（常见有效）

当出现 Wi‑Fi/网络异常、莫名其妙连不上 等情况，可尝试清理系统网络配置缓存（会丢失部分网络记录，需要重新连 Wi‑Fi）：

1）打开 Finder，前往目录（菜单：前往 -> 前往文件夹）：
/Library/Preferences/SystemConfiguration/

2）将下列文件拖进废纸篓（或先备份到别处）：
- com.apple.airport.preferences.plist
- com.apple.network.identification.plist
- com.apple.wifi.message-tracer.plist
- NetworkInterfaces.plist
- preferences.plist

3）关机并重新启动（按你的笔记：关机重启，而不是仅重启）。

提示：如果你在使用 VPN 的系统代理或虚拟网卡，先关闭它们再重启，避免网络状态被劫持。


2. 旧系统连不上 Apple 服务：设置软件更新目录（可选）

某些旧版 macOS 可能出现 Apple 服务/更新源不可用 的情况，可以通过设置 nvram 的 Software Update Catalog URL 尝试解决。

2.1 情况 A：使用 lASUCatalogURL

sudo nvram lASUCatalogURL=https://swscan.apple.com/content/catalogs/others/index-10.16seed10.16-10.15-10.14-10.13-10.12-10.11-10.10-10.9-mountainlion-lion-snowleopard.leopard.merged-1.sucatalog

2.2 情况 B：安装/恢复时报 “无法与服务器取得联系”（IASUCatalogURL）

sudo nvram IASUCatalogURL=https://swscan.apple.com/content/catalogs/others/index-10.16seed-10.16-10.15-10.14-10.13-10.12-10.11-10.10-10.9-mountainlion-lion-snowleopard-leopard.merged-1.sucatalog

说明：nvram 会修改机器的启动参数存储。仅在明确遇到该问题时使用；如后续需要恢复默认，可再查 Apple 官方/社区的恢复方式（不同系统版本略有差异）。


3. 双系统恢复 macOS（Boot Camp/Windows 清理思路）

操作顺序（偏经验流）：
1）先利用 Boot Camp 删除虚拟机/Windows 项（如有）
2）使用 磁盘工具 抹除 Windows 磁盘/分区
3）若恢复/安装过程报 无法与服务器取得联系，可参考上节 IASUCatalogURL 的处理

注意：抹除磁盘会导致数据不可恢复，务必提前备份。


4. 安装 Node.js（示例：Node 22.x）与 npm 国内镜像

4.1 安装 Node.js
- 双击 nodejs22.22.dmg 文件安装 Node.js

验证：
node -v
npm -v

4.2 配置 npm 国内镜像（两种方式）

方式 1：直接使用淘宝镜像（npmmirror）
npm config set registry https://registry.npmmirror.com/

方式 2：使用 nrm 管理镜像源
npm install -g nrm
nrm ls
nrm use taobao
nrm test

参考链接：https://cloud.tencent.com/developer/article/2483466


5. （推荐）解决 macOS 上全局 npm 权限/路径：使用 ~/.npm-global

为了避免全局安装时频繁 sudo 或权限问题，可以把全局安装目录迁到用户目录：

# 1 创建一个全局目录
mkdir -p ~/.npm-global

# 2 告诉 npm 用这个目录
npm config set prefix '~/.npm-global'

# 3 把它加进 PATH（zsh）
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc

# 如果你用 bash：
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc

# 4 重新加载（zsh）
source ~/.zshrc

验证：
which openclaw
echo $PATH


6. 使用 Git：需要更新 Xcode（命令行工具）

如果你在 macOS 上执行 git、编译依赖等遇到报错，通常需要安装/更新 Xcode Command Line Tools。

xcode-select --install


7. GitHub SSH 报错：Permission denied (publickey) 的修复流程

典型错误：
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

7.1 生成 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

默认会生成：
- 私钥：~/.ssh/id_rsa
- 公钥：~/.ssh/id_rsa.pub

7.2 将公钥添加到 GitHub
cat ~/.ssh/id_rsa.pub
GitHub：Settings -> SSH and GPG keys -> New SSH key

7.3 测试 SSH 连接
ssh -T git@github.com
正常输出示例：
Hi username! You've successfully authenticated, but GitHub does not provide shell access.

7.4 确认仓库 remote 使用 SSH（非 HTTPS）
git remote -v
git remote set-url origin git@github.com:username/repository.git

完成后再执行你需要的安装/拉取操作（例如继续 npm install 等）。


8. 安装 Homebrew（可选但很常用）

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/usr/local/bin/brew shellenv)"

验证：
brew -v


9. 安装 OpenClaw 与基础配置

9.1 安装 OpenClaw
npm install -g openclaw@latest

9.2 进入新手引导（含 daemon）
openclaw onboard --install-daemon

9.3 配置（交互式）
openclaw configure


10. 飞书插件：安装、权限 scopes、配置 appId/appSecret

10.1 插件仓库参考
https://github.com/m1heng/clawdbot-feishu?tab=readme-ov-file#%E4%B8%AD%E6%96%87

10.2 安装飞书插件
openclaw plugins install @m1heng-clawd/feishu

10.3 权限 scopes 配置示例

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

10.4 配置飞书 appId / appSecret / 启用 / 连接模式

注意：请不要在博客/仓库里写真实的 appSecret。下面是占位符示例：

openclaw config set channels.feishu.appId "cli_xxxxxxxxxxxxxxxxx"
openclaw config set channels.feishu.appSecret "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
openclaw config set channels.feishu.enabled true
openclaw config set channels.feishu.connectionMode "websocket"


11. 需要注册/准备的网站清单

1. 邮箱：https://mail.163.com/
2. VPN：https://2024.vp1.link/
3. 飞书开发平台：https://open.feishu.cn/app
4. 模型 MiniMax：https://www.minimax.io/
5. 联网搜索 skills-api（Brave Search API）：https://brave.com/search/api/


12. OpenClaw 常用命令速查

安装与引导：
npm install -g openclaw@latest
openclaw onboard --install-daemon
openclaw configure

终端启动时（环境变量）：
source ~/.zshrc

网关管理：
openclaw gateway start
openclaw gateway restart
openclaw gateway stop


13. 重启/关机前后的操作清单（强烈建议照做）

13.1 电脑关机/重启前
1）先关闭 vpn 的系统代理和虚拟网卡
2）使用命令关闭网关：
openclaw gateway stop

13.2 电脑重启后，若要重启 openclaw
1）启动终端先运行：
source ~/.zshrc
2）然后启动网关：
openclaw gateway start
或：
openclaw gateway restart


14. 最小自检清单（建议收藏）

- node -v / npm -v 正常
- npm config get registry 指向你期望的镜像
- openclaw -v 可执行，且 which openclaw 指向 ~/.npm-global/bin/openclaw（若你采用该方案）
- ssh -T git@github.com 通过（如需 GitHub）
- 飞书：appId/appSecret 已配置、scopes 正确、连接模式 websocket
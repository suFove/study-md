# OpenClaw Full macOS Setup Guide  
(Network Fix / Node / npm / Git SSH / Homebrew / Feishu Plugin / Gateway)

<p align="right">
  <a href="./README.en.md">English</a> |
  <a href="./README.zh.md">简体中文</a>
</p>

> Target audience: Developers setting up OpenClaw (including gateway & plugins) on macOS from scratch.  
> ⚠️ Never expose secrets (appSecret / SSH private key).

---

# 1. Network Troubleshooting (macOS)

If you encounter:

- Wi-Fi issues
- Network failure
- Proxy conflicts
- Virtual network adapter problems

Go to:

```
/Library/Preferences/SystemConfiguration/
```

Remove:

```
com.apple.airport.preferences.plist
com.apple.network.identification.plist
com.apple.wifi.message-tracer.plist
NetworkInterfaces.plist
preferences.plist
```

Then:

```
Shutdown → Power On
```

---

# 2. Install Node.js

Install Node.js 22.x and verify:

```bash
node -v
npm -v
```

---

# 3. Configure npm Registry

```bash
npm config set registry https://registry.npmmirror.com/
```

---

# 4. Fix Global npm Permission (Recommended)

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

# 5. Install Xcode CLI Tools

```bash
xcode-select --install
```

---

# 6. Fix GitHub SSH (publickey error)

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
ssh -T git@github.com
```

---

# 7. Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

# 8. Install OpenClaw

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
openclaw configure
```

---

# 9. Install Feishu Plugin

```bash
openclaw plugins install @m1heng-clawd/feishu
```

Configure:

```bash
openclaw config set channels.feishu.appId "cli_xxxxx"
openclaw config set channels.feishu.appSecret "xxxxxx"
openclaw config set channels.feishu.enabled true
openclaw config set channels.feishu.connectionMode "websocket"
```

---

# 10. Gateway Management

```bash
openclaw gateway start
openclaw gateway restart
openclaw gateway stop
```

---

# ✔ Final Checklist

```bash
node -v
npm -v
which openclaw
ssh -T git@github.com
```

---

You now have:

- Clean macOS network
- Proper Node/npm setup
- SSH working
- Homebrew installed
- OpenClaw daemon running
- Feishu WebSocket bot ready

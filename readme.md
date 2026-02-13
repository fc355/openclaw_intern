---

# 🦞 OpenClaw Installation Guide

คู่มือนี้อธิบายขั้นตอนการติดตั้ง **OpenClaw** และเชื่อมต่อกับ:

* LLM ภายในองค์กร (ผ่าน LiteLLM)
* Mattermost (Bot Integration)

---

# 📦 Prerequisites

* Ubuntu / Debian Linux
* Node.js v22+
* npm
* สิทธิ์ sudo

---

# 🚀 Step 1: ตรวจสอบและติดตั้ง Node.js

ตรวจสอบก่อน:

```bash
node -v
npm -v
```

ถ้ายังไม่มี ให้ติดตั้ง:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

ตรวจสอบอีกครั้ง:

```bash
node -v
npm -v
```

---

# 🦞 Step 2: ติดตั้ง OpenClaw

```bash
sudo npm install -g openclaw
```

ตรวจสอบเวอร์ชัน:

```bash
openclaw --version
```

---

# ⚙️ Step 3: Initial Setup (Onboard)

```bash
openclaw onboard
```

* เลือก provider (เช่น OpenAI / Qwen)
* หากใช้ LiteLLM ภายในองค์กร สามารถ skip provider ภายนอกได้

ระบบจะสร้าง config ที่:

```
~/.openclaw/openclaw.json
```

---

# 🔌 Step 4: ติดตั้ง Gateway Service

ติดตั้ง daemon service:

```bash
openclaw onboard --install-daemon
```
 I understand this is powerful and inherently risky. Continue?
 -yes

 Onboarding mode
 -QuickStart

* ถ้ามันให้เลือก Model/Provider แนะนำให้ skip เพราะเราจะเข้าไปเพิ่มใน openclaw.json ครับ


ตรวจสอบสถานะ:

```bash
openclaw gateway status
```

---

# 🧠 Step 5: เชื่อมต่อ LLM ภายในองค์กร (ผ่าน LiteLLM) สามารถดู openclaw.json ใน Git เพื่อเป็นตัวอย่างได้ครับ

แก้ไขไฟล์ config:

```bash
nano ~/.openclaw/openclaw.json
```

เพิ่ม provider แบบนี้:

```json
"models": {
    "providers": {
      "litellm": {
        "baseUrl": "{IPLLM-Target:Port}/v1",
        "apiKey": "{API-KEY}",
        "api": "openai-completions",
        "headers": {
          "x-litellm-api-key": "{API-KEY}"
        },
        "authHeader": false,
        "models": [
          {
            "id": "gemma-3-27b-it",
            "name": "Gemma 3 27B IT (LiteLLM)",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "litellm/gemma-3-27b-it"
      },
      "models": {
        "litellm/gemma-3-27b-it": {
          "alias": "gemma"
        }
      },
      "workspace": "/home/intern/.openclaw/workspace",
      "compaction": {
        "mode": "safeguard"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  },
  "tools": {
    "byProvider": {
      "litellm/gemma-3-27b-it": {
        "deny": ["*"]
      }
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto"
```


หลังแก้ไขเสร็จ restart gateway:

```bash
openclaw gateway restart
```

---

# 💬 Step 6: เชื่อมต่อ Mattermost

## 6.1 ติดตั้ง Plugin

```bash
openclaw plugins install ./extensions/mattermost
openclaw plugins enable mattermost
```

---

## 6.2 เพิ่ม Mattermost Config

แก้ไฟล์:

```bash
nano ~/.openclaw/openclaw.json
```

เพิ่ม:

```json
"channels": {
  "mattermost": {
    "enabled": true,
    "baseUrl": "https://your-mattermost-url",
    "botToken": "YOUR_BOT_TOKEN",
    "dmPolicy": "pairing",
    "chatmode": "onchar",
    "oncharPrefixes": [">"]
  }
}
```

คำอธิบายค่า:

* `baseUrl` → URL Mattermost
* `botToken` → ได้จาก Integrations → Bot Account
* `dmPolicy: pairing` → ต้อง pair bot ก่อนเริ่มใช้งาน
* `chatmode: onchar` → ใช้ prefix เรียก bot
* `oncharPrefixes` → เช่น `>` เพื่อ trigger bot

หลังแก้ไขเสร็จ:

```bash
openclaw gateway restart
```

---

# 🔍 ตรวจสอบสถานะระบบ

```bash
openclaw gateway status
```

ถ้าทำงานปกติ:

```
Gateway running
```

---

# 🧱 Architecture Overview

```
User → Mattermost → OpenClaw Gateway → LiteLLM → Internal LLM
```

---

# 🛠 Troubleshooting

## 🔹 Gateway ไม่ทำงาน

```bash
openclaw gateway logs
```

## 🔹 ตรวจสอบ config

```bash
cat ~/.openclaw/openclaw.json
```

## 🔹 Restart service

```bash
openclaw gateway restart
```

---

# 🔐 Security Recommendations

* ใช้ HTTPS สำหรับ Mattermost
* อย่า commit `openclaw.json` ที่มี API Key ลง Git
* จำกัดสิทธิ์ bot account ให้เหมาะสม
* ใช้ firewall จำกัดการเข้าถึง Gateway

---

# 🎯 Completed

หลังจากติดตั้งครบทุกขั้นตอน:

* OpenClaw พร้อมใช้งาน
* เชื่อมต่อ LLM ภายในองค์กรได้
* ใช้งานผ่าน Mattermost ได้ด้วย prefix เช่น `>`

---

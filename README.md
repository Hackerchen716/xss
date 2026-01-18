# 🔥 XSS Labs & CTF Writeups

> XSS 漏洞研究、CTF 题解、Payload 分析、绕过技巧、PoC 合集

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![XSS](https://img.shields.io/badge/Security-XSS-red.svg)](https://owasp.org/www-community/attacks/xss/)

---

## 📚 目录

| 平台                | 题目                   | 难度 | 核心考点                   | 状态 |
| ------------------- | ---------------------- | ---- | -------------------------- | ---- |
| [njhack](./njhack/) | [0x0A](./njhack/0x0A/) | ⭐⭐⭐  | HTML 实体漏分号 + 域名绕过 | ✅    |

---

## 🎯 仓库结构

```
xss/
├── README.md                 # 本文件
├── LICENSE                   # MIT License
└── njhack/                   # njhack 平台题目
    └── 0x0A/                 # 题目编号
        ├── README.md         # 详细 Writeup
        ├── assets/           # 截图证据
        └── payload/          # Payload 文件
```

---

## 🔬 已完成题解

### njhack - xss1.njhack.xyz

#### [0x0A - JS Payload 错误解析 / HTML 实体吞字符绕过](./njhack/0x0A/)

**核心漏洞**: 服务端把 `/` 替换为 `&#x2f` 但漏了分号 `;`，导致浏览器把 `&#x2fa` 当成一个 Unicode 字符 `˺`

**Payload**:

```
http://www.segmentfault.com.localhost:8000/z.js
```

**关键点**:

- 利用 `.localhost` 伪 TLD 绕过域名前缀校验
- 文件名不能以十六进制字符开头（`0-9a-f`），否则会被实体吞掉

---

## 📖 知识体系

### XSS 类型

- **反射型 XSS**: URL 参数注入
- **存储型 XSS**: 持久化到数据库
- **DOM XSS**: 前端 JS 处理不当
- **mXSS**: 浏览器解析差异

### 常见绕过技巧

- HTML 实体编码（`&#x`、`&#`、命名实体）
- URL 编码（`%xx`）
- Unicode 编码（`\uXXXX`）
- 大小写混淆
- 标签/属性变形
- 注释截断
- 协议绕过（`javascript:`、`data:`）

### 浏览器安全机制

- CSP (Content Security Policy)
- X-XSS-Protection
- HttpOnly Cookie
- SameSite Cookie
- Trusted Types

---

## 🛠️ 常用工具

| 工具                                       | 用途                   |
| ------------------------------------------ | ---------------------- |
| [Burp Suite](https://portswigger.net/burp) | 抓包、重放、扫描       |
| [XSS Hunter](https://xsshunter.com/)       | Blind XSS 回连         |
| [BeEF](https://beefproject.com/)           | 浏览器利用框架         |
| Chrome DevTools                            | 调试、Network、Console |

---

## 📚 学习资源

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/cross-site-scripting) - 免费 XSS 实验室
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [HackTricks - XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting)
- [PayloadsAllTheThings - XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)

---

## ⚠️ 免责声明

本仓库仅供安全研究和学习交流使用。请勿将这些技术用于非法目的。在进行任何安全测试之前，请确保已获得适当的授权。

---

## 📝 License

[MIT License](./LICENSE)

---

**Author**: Hackerchen716  
**Blog**: [CYBERLOG](https://hackerchen.com)

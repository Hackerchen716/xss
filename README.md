# 🔥 XSS Labs & CTF Writeups

> XSS 漏洞研究、CTF 题解、Payload 分析、绕过技巧、PoC 合集

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![XSS](https://img.shields.io/badge/Security-XSS-red.svg)](https://owasp.org/www-community/attacks/xss/)

---

## 📚 靶场目录

### 🎯 xss1.njhack.xyz

国内 XSS 练习靶场，共 19 关（0x00 - 0x12），难度递进。

| 关卡 | 难度 | 核心考点 | Writeup |
|------|------|---------|---------|
| 0x00 | ⭐ | 无过滤 | [查看](./xss1-njhack/README.md#-0x00---无过滤入门热身) |
| 0x01 | ⭐ | textarea 闭合 | [查看](./xss1-njhack/README.md#-0x01---textarea-标签逃逸) |
| 0x02 | ⭐ | input 属性逃逸 | [查看](./xss1-njhack/README.md#-0x02---input-标签属性逃逸) |
| 0x03 | ⭐⭐ | 括号过滤 → ES6 模板字符串 | [查看](./xss1-njhack/README.md#-0x03---过滤括号-) |
| 0x04 | ⭐⭐ | SVG + HTML 实体编码 | [查看](./xss1-njhack/README.md#-0x04---过滤括号和反引号-) |
| 0x05 | ⭐⭐ | HTML 注释 `--!>` | [查看](./xss1-njhack/README.md#-0x05---html-注释逃逸) |
| 0x06 | ⭐⭐ | 换行绕过正则 | [查看](./xss1-njhack/README.md#-0x06---事件属性注入换行绕过) |
| 0x07 | ⭐⭐ | 不闭合标签 | [查看](./xss1-njhack/README.md#-0x07---标签剥离绕过) |
| 0x08 | ⭐⭐ | style 标签逃逸 | [查看](./xss1-njhack/README.md#-0x08---style-标签逃逸) |
| 0x09 | ⭐⭐ | URL 前缀匹配漏洞 | [查看](./xss1-njhack/README.md#-0x09---url-正则验证前缀匹配漏洞) |
| **0x0A** | ⭐⭐⭐ | **HTML 实体漏分号 + localhost** | [**详细**](./xss1-njhack/0x0A/) |
| 0x0B | ⭐⭐⭐ | 大写转换 → 外部脚本 | [查看](./xss1-njhack/README.md#-0x0b---大写转换绕过外部脚本加载) |
| 0x0C | ⭐⭐⭐ | 双写绕过 | [查看](./xss1-njhack/README.md#-0x0c---script-关键字过滤--大写转换) |
| 0x0D | ⭐⭐⭐ | JS 注释 + `-->` | [查看](./xss1-njhack/README.md#-0x0d---javascript-注释逃逸) |
| 0x0E | ⭐⭐⭐ | Unicode `ſ` → `S` | [查看](./xss1-njhack/README.md#-0x0e---字母标签过滤--大写转换unicode-魔法) |
| 0x0F | ⭐⭐⭐ | HTML 属性解码机制 | [查看](./xss1-njhack/README.md#-0x0f---html-属性中的实体解码内鬼机制) |
| 0x10 | ⭐⭐ | JS 变量注入 | [查看](./xss1-njhack/README.md#-0x10---javascript-变量赋值注入) |
| 0x11 | ⭐⭐⭐ | 退格符吃反斜杠 | [查看](./xss1-njhack/README.md#-0x11---javascript-url-协议注入) |
| 0x12 | ⭐⭐⭐ | 反斜杠逃逸 | [查看](./xss1-njhack/README.md#-0x12---反斜杠逃逸) |

📖 **完整教程**：[xss1-njhack/README.md](./xss1-njhack/README.md)

---

### 🎯 PortSwigger Web Security Academy

> *Coming Soon...*

### 🎯 XSS-Game by Google

> *Coming Soon...*

### 🎯 prompt.ml

> *Coming Soon...*

### 🎯 其他靶场

> *持续更新中...*

---

## 🗂️ 仓库结构

```
xss/
├── README.md                      # 本文件（仓库总览）
├── LICENSE
│
├── xss1-njhack/                   # njhack 靶场
│   ├── README.md                  # 完整通关教程（19关）
│   ├── 0x0A/                      # 重点关卡单独目录
│   │   ├── README.md              # 详细 Writeup
│   │   ├── assets/                # 截图
│   │   └── payload/               # Payload 文件
│   └── assets/                    # 通用截图
│
├── portswigger/                   # PortSwigger 靶场（待添加）
│   ├── README.md
│   └── ...
│
├── xss-game/                      # Google XSS Game（待添加）
│   └── ...
│
└── cheatsheet/                    # 速查表
    ├── payloads.md                # 常用 Payload
    ├── bypass-techniques.md       # 绕过技巧
    └── encoding.md                # 编码对照表
```

---

## 📖 知识体系

### XSS 类型

| 类型 | 描述 | 危害程度 |
|------|------|---------|
| 反射型 | URL 参数注入，需诱导点击 | ⭐⭐ |
| 存储型 | 持久化到数据库，影响所有用户 | ⭐⭐⭐ |
| DOM 型 | 纯前端 JS 处理不当 | ⭐⭐ |
| mXSS | 浏览器解析差异导致 | ⭐⭐⭐ |

### 绕过技巧速览

| 技巧 | 适用场景 | 示例 |
|------|---------|------|
| 标签闭合 | 被困在标签内 | `</textarea><script>` |
| 属性逃逸 | 被困在属性内 | `"><script>` |
| 大小写混淆 | 黑名单过滤 | `<ScRiPt>` |
| 双写绕过 | 单次替换过滤 | `<scrscriptipt>` |
| HTML 实体 | 括号等字符过滤 | `&#40;&#41;` |
| Unicode 变形 | 字母过滤 + 大写转换 | `ſ` → `S` |
| 换行打断 | 正则单行匹配 | `on\nmouseover=` |
| 注释逃逸 | JS/HTML 注释 | `-->`, `--!>` |
| 编码绕过 | 字符过滤 | `%0a`, `%08` |

### 常用工具

| 工具 | 用途 | 链接 |
|------|------|------|
| JSFuck | 非字母数字编码 | [jsfuck.com](http://jsfuck.com/) |
| CyberChef | 编码解码 | [gchq.github.io](https://gchq.github.io/CyberChef/) |
| Burp Suite | 抓包测试 | [portswigger.net](https://portswigger.net/burp) |
| XSS Hunter | Blind XSS | [xsshunter.com](https://xsshunter.com/) |

---

## 📚 学习资源

- [PortSwigger XSS Labs](https://portswigger.net/web-security/cross-site-scripting) - 免费在线靶场
- [OWASP XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) - 防御指南
- [HackTricks XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting) - 技巧大全
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) - Payload 集合

---

## ⚠️ 免责声明

本仓库仅供安全研究和学习交流使用。请勿将这些技术用于非法目的。在进行任何安全测试之前，请确保已获得适当的授权。

---

## 📝 License

[MIT License](./LICENSE)

---

**Author**: Hackerchen  
**Blog**: [CYBERLOG](https://hackerchen.com)

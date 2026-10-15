
# DOM 树详解

DOM 树是理解网页结构的基础，也是 DevTools 和安全学习的关键概念。我把它讲透。

---

## 一、什么是 DOM？

**DOM = Document Object Model（文档对象模型）**

简单说：**浏览器把 HTML 文档解析成一棵"树"，用 JavaScript 可以操作这棵树。**

- **HTML** 是你写的文本
- **DOM** 是浏览器解析后内存里的树结构
- **JavaScript** 通过 DOM API 操作页面

```
你写的 HTML  →  浏览器解析  →  DOM 树（内存）  →  渲染成页面
```

---

## 二、为什么叫"树"？

因为 HTML 的嵌套结构天然就是树形：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>我的页面</title>
  </head>
  <body>
    <h1>标题</h1>
    <p>段落</p>
  </body>
</html>
```

解析成 DOM 树：

```
document
  └── html
       ├── head
       │    └── title
       │         └── "我的页面"
       └── body
            ├── h1
            │    └── "标题"
            └── p
                 └── "段落"
```

**树的特点**：

- 有**根**（document）
- 每个节点有**父节点**、**子节点**、**兄弟节点**
- 层层嵌套

---

## 三、DOM 节点的类型

不是只有标签，所有东西都是节点：

| 节点类型           | 例子               | nodeType |
| ------------------ | ------------------ | -------- |
| **元素节点** | `<div>`、`<p>` | 1        |
| **文本节点** | "标题" 这段文字    | 3        |
| **属性节点** | `class="box"`    | 2        |
| **注释节点** | `<!-- 注释 -->`  | 8        |
| **文档节点** | `document`       | 9        |

**例子**：

```html
<div class="box">你好<!-- 注释 --></div>
```

DOM 树：

```
div (元素节点)
├── 属性: class="box"  (属性节点)
├── "你好"              (文本节点)
└── <!-- 注释 -->       (注释节点)
```

---

## 四、节点之间的关系

```html
<html>
  <body>
    <div>
      <h1>标题</h1>
      <p>段落</p>
    </div>
  </body>
</html>
```

以 `<div>` 为中心：

```
        body
          │
        div  ←── 当前节点
        / \
      h1   p
```

| 关系           | 节点    | 属性                                       |
| -------------- | ------- | ------------------------------------------ |
| 父节点         | body    | `div.parentNode`                         |
| 子节点         | h1, p   | `div.childNodes`                         |
| 第一个子节点   | h1      | `div.firstChild`                         |
| 最后一个子节点 | p       | `div.lastChild`                          |
| 兄弟节点       | h1 ↔ p | `h1.nextSibling` / `p.previousSibling` |

---

## 五、DOM API：用 JS 操作树

### 1. 查找节点

```js
// 按选择器查
document.querySelector('.box')        // 第一个匹配
document.querySelectorAll('p')        // 所有匹配（NodeList）

// 按 ID
document.getElementById('app')

// 按类名
document.getElementsByClassName('item')

// 按标签
document.getElementsByTagName('div')

// 关系查找
el.parentNode
el.children          // 只含元素节点
el.childNodes        // 含所有节点（文本、注释）
el.firstElementChild
el.nextElementSibling
```

### 2. 创建节点

```js
const div = document.createElement('div');
div.textContent = '新内容';
div.className = 'box';
```

### 3. 插入节点

```js
parent.appendChild(div)              // 末尾追加
parent.insertBefore(div, refNode)    // 在 refNode 前插入
parent.prepend(div)                  // 开头插入
el.after(div)                        // 在 el 之后
el.before(div)                       // 在 el 之前
```

### 4. 删除节点

```js
parent.removeChild(child)
el.remove()                          // 现代写法
```

### 5. 修改节点

```js
el.textContent = '新文本'             // 改文本
el.innerHTML = '<b>加粗</b>'         // 改 HTML（危险！）
el.setAttribute('data-id', '1')      // 改属性
el.style.color = 'red'               // 改样式
el.classList.add('active')           // 加类
```

---

## 六、DOM 树在 DevTools 里长什么样

打开 DevTools → Elements 面板，你看到的就是 DOM 树：

```
▼ <html>
  ▼ <head>
      <title>我的页面</title>
    </head>
  ▼ <body>
      <h1>标题</h1>
      <p>段落</p>
    </body>
</html>
```

**注意**：这里显示的是**解析后的 DOM**，不一定和源码完全一样。

**为什么？**

1. 浏览器会**自动补全**标签（比如 `<tbody>`）
2. JavaScript **动态修改**后的结果
3. 错误的 HTML 会被**纠正**

**验证**：

右键页面 → 查看网页源代码（`Ctrl+U`）看的是**原始 HTML**；
DevTools 的 Elements 看的是**当前 DOM**。

两者可能不同。

---

## 七、DOM 树 vs HTML 源码

|        | HTML 源码 | DOM 树            |
| ------ | --------- | ----------------- |
| 是什么 | 文本文件  | 内存中的对象树    |
| 谁生成 | 你写的    | 浏览器解析后      |
| 会变吗 | 不变      | JS 可动态改       |
| 在哪看 | Ctrl+U    | DevTools Elements |
| 用途   | 传输      | 渲染 + 操作       |

**例子**：

HTML 源码：

```html
<ul id="list"></ul>
```

JS 执行后：

```js
document.getElementById('list').innerHTML = '<li>项目1</li>';
```

DOM 树变成：

```
ul#list
  └── li
       └── "项目1"
```

但源码里 `<ul>` 还是空的。

---

## 八、DOM 树与渲染的关系

浏览器渲染页面的流程：

```
HTML → 解析 → DOM 树
CSS  → 解析 → CSSOM 树
                │
                ▼
          Render 树（渲染树）
                │
                ▼
          布局（Layout）
                │
                ▼
          绘制（Paint）
                │
                ▼
          合成（Composite）
                │
                ▼
              屏幕
```

**关键**：

- **DOM 树**：页面结构
- **CSSOM 树**：样式规则
- **Render 树**：只包含可见元素（`display:none` 不在里面）
- 改 DOM 会触发**重排（reflow）**或**重绘（repaint）**

---

## 九、DOM 树与网络安全

这是重点，DOM 树是很多漏洞的根源。

### 1. DOM 型 XSS

**特点**：恶意数据不经过服务器，直接在浏览器里被 JS 插入 DOM。

**例子**：

```js
// 危险代码
const name = location.hash.slice(1);   // 从 URL 取
document.getElementById('welcome').innerHTML = '你好，' + name;
```

攻击：

```
https://site.com/#<img src=x onerror=alert(1)>
```

`innerHTML` 把恶意 HTML 插入 DOM，触发 XSS。

**为什么危险**：

- `innerHTML` 会解析 HTML
- `<script>` 不执行，但 `<img onerror>` 会
- 数据来自 URL，服务器看不到

**安全写法**：

```js
el.textContent = name;   // 只当文本，不解析
```

### 2. DOM Clobbering（DOM 覆盖）

利用 HTML 元素覆盖全局变量。

```html
<form id="config">
  <input name="apiUrl" value="https://evil.com">
</form>
```

```js
// 本意是用全局 config 对象
fetch(config.apiUrl)   // 结果被 form 覆盖，请求发到 evil.com
```

**原理**：`id` 和 `name` 会自动成为 `window` 的属性，覆盖原有变量。

### 3. 原型污染

通过 DOM 操作污染 `Object.prototype`。

```js
// 如果库里有这种合并逻辑
function merge(target, source) {
  for (let key in source) {
    target[key] = source[key];
  }
}
merge({}, JSON.parse('{"__proto__":{"isAdmin":true}}'));
// 现在所有对象都有 isAdmin = true
```

### 4. DOM 断点调试（安全分析）

DevTools → Elements → 右键元素 → Break on：

- **Subtree modifications**：子节点变化时暂停
- **Attribute modifications**：属性变化时暂停
- **Node removal**：节点被删除时暂停

**用途**：追踪恶意脚本何时、如何修改 DOM。

---

## 十、一个完整例子

### HTML

```html
<!DOCTYPE html>
<html>
<head>
  <title>DOM 演示</title>
</head>
<body>
  <div id="app">
    <h1>你好</h1>
  </div>
  <script>
    // 1. 查找
    const app = document.getElementById('app');

    // 2. 创建
    const p = document.createElement('p');
    p.textContent = '这是新段落';

    // 3. 插入
    app.appendChild(p);

    // 4. 修改
    document.querySelector('h1').style.color = 'blue';

    // 5. 删除
    // app.removeChild(p);
  </script>
</body>
</html>
```

### 执行后的 DOM 树

```
document
  └── html
       ├── head
       │    └── title → "DOM 演示"
       └── body
            ├── div#app
            │    ├── h1 → "你好"（蓝色）
            │    └── p → "这是新段落"
            └── script
```

在 DevTools Elements 里能看到这些变化。

---

## 十一、常见面试/考点

| 问题                             | 答案要点                             |
| -------------------------------- | ------------------------------------ |
| DOM 是什么                       | 文档对象模型，HTML 的内存树结构      |
| DOM 树根节点                     | `document`                         |
| `children` vs `childNodes`   | 前者只含元素，后者含所有节点         |
| `innerHTML` vs `textContent` | 前者解析 HTML，后者纯文本            |
| 为什么 DOM 操作慢                | 触发重排重绘，跨 JS/渲染引擎         |
| 如何优化 DOM 操作                | 批量更新、DocumentFragment、虚拟 DOM |
| DOM 型 XSS 原理                  | 数据未过滤直接插入 DOM               |

---

## 十二、一句话总结

> **DOM 树是浏览器把 HTML 解析后在内存里构建的树形结构**，JavaScript 通过 DOM API 操作它，浏览器根据它渲染页面。理解 DOM 树，是学前端、调样式、做安全（尤其 XSS）的基础。

---

## 十三、动手练习

1. **看树**：打开任意网站，DevTools Elements，展开看看结构
2. **改树**：Console 里执行 `document.body.style.background = 'red'`
3. **建树**：写一段 JS，动态创建一个列表
4. **安全**：用 `innerHTML` 和 `textContent` 分别插入 `<script>alert(1)</script>`，看区别
5. **调试**：在 Elements 里给某元素加 DOM 断点，看什么时候被修改

---

**DOM 型 XSS 的完整利用链**，还是 **DOM 操作性能优化**（重排重绘、虚拟 DOM）

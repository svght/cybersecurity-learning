# 浏览器开发者工具（DevTools）完全学习指南

浏览器开发者工具是前端和 Web 开发最重要的调试利器。下面我按**面板**逐个讲透，每个都配场景和实操。

---

## 目录

- [一、如何打开 DevTools](#一如何打开-devtools)
- [二、Elements（元素）面板](#二elements元素面板)
- [三、Console（控制台）](#三console控制台)
- [四、Sources（源代码）](#四sources源代码)
- [五、Network（网络）](#五network网络最重要)
- [六、Application（应用）](#六application应用)
- [七、Performance（性能）](#七performance性能)
- [八、Lighthouse（灯塔）](#八lighthouse灯塔)
- [九、其他实用面板](#九其他实用面板)
- [十、实战调试场景](#十实战调试场景)
- [十一、高效技巧](#十一高效技巧)

---

# 一、如何打开 DevTools

| 方式   | 操作                                                 |
| ------ | ---------------------------------------------------- |
| 快捷键 | `F12` 或 `Ctrl+Shift+I`（Mac: `Cmd+Option+I`） |
| 右键   | 页面任意处 → "检查"                                 |
| 菜单   | 浏览器菜单 → 更多工具 → 开发者工具                 |

**布局调整**：

- 点右上角 `⋮` → Dock side，可切换停靠位置（底部/右侧/独立窗口）
- `Ctrl+Shift+M`：切换设备模拟（手机/平板视图）

---

# 二、Elements（元素）面板

**用途**：查看和修改页面的 HTML 结构和 CSS 样式。

## 2.1 左侧：DOM 树 (DOM=Document Object Model)

**浏览器把 HTML 文档解析成一棵"树"，用 JavaScript 可以操作这棵树。**

```
<body>
  <div class="container">
    <h1>标题</h1>
    <p>段落</p>
  </div>
</body>
```

**常用操作**：

| 操作         | 方法                                             |
| ------------ | ------------------------------------------------ |
| 展开/折叠    | 点三角箭头                                       |
| 编辑 HTML    | 双击标签或文字                                   |
| 删除元素     | 选中 → `Delete`                               |
| 复制元素     | 右键 → Copy → Copy element                     |
| 复制选择器   | 右键 → Copy → Copy selector                    |
| 强制元素状态 | 右键 → Force state（:hover / :active / :focus） |
| 滚动到视图   | 右键 → Scroll into view                         |

**小技巧**：

- 点左上角"箭头图标"（`Ctrl+Shift+C`），再点页面元素，直接定位
- 鼠标悬停在 DOM 节点上，页面对应区域会高亮

## 2.2 右侧：Styles（样式）

显示当前元素应用的所有 CSS 规则。

**能做什么**：

```css
/* 直接修改，实时生效 */
.container {
  background: red;      /* 改颜色 */
  width: 500px;         /* 改尺寸 */
  display: flex;        /* 加新属性 */
}
```

| 功能      | 说明                                         |
| --------- | -------------------------------------------- |
| 实时编辑  | 点属性值直接改，页面立即变化                 |
| 勾选/取消 | 左侧复选框可临时禁用某条规则                 |
| 查看来源  | 右上角显示规则来自哪个文件哪一行             |
| 盒模型图  | 底部彩色图显示 margin/border/padding/content |
| Computed  | 查看最终计算后的样式                         |
| 过滤      | 搜索框输入属性名快速定位                     |

**盒模型示意**：

```
┌─────────────────────────────┐
│         margin (橙)          │
│  ┌───────────────────────┐  │
│  │    border (黄)         │  │
│  │  ┌─────────────────┐  │  │
│  │  │  padding (绿)    │  │  │
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  content  │  │  │  │
│  │  │  └───────────┘  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

## 2.3 常见用途

- **调样式**：改颜色、间距、布局，满意后复制回代码
- **找问题**：为什么这个元素不显示？看 `display: none` 还是被覆盖
- **看继承**：哪些样式是从父元素继承来的
- **查层级**：用 `z-index` 调试遮挡问题

---

# 三、Console（控制台）

**用途**：执行 JS、查看日志、调试错误。

## 3.1 查看日志

```js
console.log('普通日志');
console.info('信息');
console.warn('警告');     // 黄色
console.error('错误');    // 红色
console.debug('调试');

// 分组
console.group('用户信息');
console.log('姓名: 张三');
console.log('年龄: 18');
console.groupEnd();

// 表格
console.table([
  { name: '张三', age: 18 },
  { name: '李四', age: 20 },
]);

// 计时
console.time('循环');
for (let i = 0; i < 100000; i++) {}
console.timeEnd('循环');  // 循环: 2.34ms

// 断言
console.assert(1 === 2, '这不对');  // 条件为 false 时输出

// 计数
console.count('点击');  // 每调用一次 +1

// 追踪调用栈
console.trace('看看谁调用的');
```

## 3.2 直接执行 JS

在 Console 输入框里可以：

```js
// 计算
1 + 1                    // 2

// 操作 DOM
document.querySelector('h1').textContent = '新标题'

// 发请求
fetch('/api/users').then(r => r.json()).then(console.log)

// 查看变量
window.location.href
document.cookie

// 引入库（临时）
const script = document.createElement('script');
script.src = 'https://cdn.jsdelivr.net/npm/jquery';
document.head.appendChild(script);
```

## 3.3 Console 的 `$` 快捷方式

| 命令               | 等价于                        |
| ------------------ | ----------------------------- |
| `$('selector')`  | `document.querySelector`    |
| `$$('selector')` | `document.querySelectorAll` |
| `$0`             | 当前 Elements 选中的元素      |
| `$1` ~ `$4`    | 之前选中的元素                |
| `$_`             | 上一个表达式的结果            |
| `$x('//xpath')`  | XPath 查询                    |

**例子**：

```js
$0.style.background = 'red'   // 给当前选中元素加背景
$$('a').length                // 页面上有多少个链接
```

## 3.4 日志过滤

Console 顶部工具栏：

- **日志级别**：Verbose / Info / Warnings / Errors
- **过滤框**：输入关键字
- **Preserve log**：页面跳转后保留日志（调试登录跳转很有用）
- **Settings**：隐藏网络日志、显示时间戳

---

# 四、Sources（源代码）

**用途**：查看源码、打断点、调试 JS。

## 4.1 面板结构

```
┌─────────────┬──────────────────┬─────────────┐
│ 文件树       │   代码编辑器      │  调试面板    │
│ (Page/Files)│                  │  (Breakpoints)│
│             │                  │             │
│ - index.html│  1 function...   │  Watch      │
│ - app.js    │  2 ...           │  Call Stack │
│ - style.css │  3 ...           │  Scope      │
└─────────────┴──────────────────┴─────────────┘
```

## 4.2 打断点（Breakpoint）

**为什么**：让代码运行到某一行暂停，方便查看变量。

**方式 1：行断点**

点代码行号，出现蓝色标记。

**方式 2：条件断点**

右键行号 → Add conditional breakpoint：

```js
i === 5    // 只有 i 等于 5 时才暂停
```

**方式 3：DOM 断点**

Elements 面板 → 右键元素 → Break on：

- Subtree modifications（子节点变化）
- Attribute modifications（属性变化）
- Node removal（节点被删除）

**方式 4：XHR/Fetch 断点**

Sources → XHR/Fetch Breakpoints → 添加 URL 关键字，请求时暂停。

**方式 5：事件断点**

Sources → Event Listener Breakpoints → 勾选事件类型（如 click）。

## 4.3 调试操作

| 按钮          | 快捷键            | 作用                   |
| ------------- | ----------------- | ---------------------- |
| ▶️ 继续     | `F8`            | 运行到下一个断点       |
| ⤵️ 单步跳过 | `F10`           | 执行下一行，不进入函数 |
| ⬇️ 单步进入 | `F11`           | 进入函数内部           |
| ⬆️ 单步跳出 | `Shift+F11`     | 跳出当前函数           |
| 🔄 重启       | `Ctrl+Shift+F5` | 重新运行               |

## 4.4 调试面板

**Watch**：添加想监控的变量表达式

```
user.name
items.length
```

**Call Stack**：当前调用栈，点某层可跳转查看

**Scope**：当前作用域的变量

- Local：局部变量
- Closure：闭包变量
- Global：全局变量

## 4.5 常用调试技巧

```js
// 1. 代码里写 debugger，运行到此处自动暂停
function foo() {
  debugger;
  console.log('这里会暂停');
}

// 2. 用 console.log 打印对象时，引用会变，用 JSON 深拷贝
console.log(JSON.parse(JSON.stringify(obj)));

// 3. 报错时点开 Call Stack，逐层看哪里出错
```

---

# 五、Network（网络）【最重要】

**用途**：查看所有网络请求，分析性能。

## 5.1 面板结构

```
┌──────────────────────────────────────────────────┐
│ 过滤器: All | Fetch/XHR | JS | CSS | Img | Media │
├──────────────────────────────────────────────────┤
│ Name          Status  Type   Size   Time         │
│ ──────────────────────────────────────────────── │
│ index.html    200     doc    1.2KB  45ms         │
│ app.js        200     script 50KB   120ms        │
│ api/users     200     fetch  2KB    230ms        │
│ logo.png      304     img    0B     10ms         │
└──────────────────────────────────────────────────┘
```

## 5.2 每列含义

| 列        | 含义                                  |
| --------- | ------------------------------------- |
| Name      | 请求的文件名/URL                      |
| Status    | HTTP 状态码（200/304/404/500）        |
| Type      | 资源类型（document/script/fetch/img） |
| Initiator | 谁发起的请求                          |
| Size      | 传输大小 / 实际大小                   |
| Time      | 总耗时                                |
| Waterfall | 时间线瀑布图                          |

## 5.3 过滤和搜索

- **All / Fetch/XHR / JS / CSS / Img / Media / Font / Doc / WS / Manifest**
- **过滤框**：输入关键字
- **高级语法**：
  - `domain:api.example.com`
  - `status-code:404`
  - `larger-than:100k`
  - `method:POST`
  - `mime-type:application/json`
  - `-` 取反，如 `-domain:localhost`

## 5.4 点开请求看详情

**Headers**：请求头、响应头、状态码、URL

```
General:
  Request URL: https://api.example.com/users/1
  Request Method: GET
  Status Code: 200 OK

Request Headers:
  Authorization: Bearer xxx
  Content-Type: application/json

Response Headers:
  Content-Type: application/json
  Cache-Control: max-age=3600
```

**Payload**：POST/PUT 请求发送的数据

**Preview**：格式化预览（JSON 可展开）

**Response**：原始响应内容

**Timing**：请求各阶段耗时

```
Queued:        排队时间
Stalled:       等待可用连接
DNS Lookup:    DNS 解析
Initial connection: TCP 握手
SSL:           TLS 握手
Request sent:  发送请求
Waiting (TTFB): 等待服务器响应
Content Download: 下载内容
```

**Cookies**：请求/响应携带的 Cookie

## 5.5 关键功能

**1. 模拟慢网速**

顶部下拉（No throttling）→ 选 Fast 3G / Slow 3G / 自定义。

**2. 禁用缓存**

勾选 "Disable cache"，刷新时强制重新请求（调试 304 缓存问题）。

**3. 保留请求**

勾选 "Preserve log"，页面跳转后请求不消失（调试登录跳转）。

**4. 复制请求**

右键请求 → Copy → Copy as cURL / Copy as fetch

**cURL 示例**：

```bash
curl 'https://api.example.com/users/1' \
  -H 'Authorization: Bearer xxx' \
  -H 'Content-Type: application/json'
```

**fetch 示例**：

```js
fetch('https://api.example.com/users/1', {
  headers: { 'Authorization': 'Bearer xxx' },
});
```

**5. 重放请求**

右键 → Replay XHR（只对 XHR/fetch 有效）。

**6. 阻塞请求**

右键 → Block request URL，模拟资源被墙。

**7. 查看 WebSocket**

选 WS 过滤器，点开可看每一帧消息。

## 5.6 分析性能

**看瀑布图**：

- 请求越多、并行度越低，总时间越长
- 关注慢请求（Time 列排序）

**常见优化点**：

| 现象     | 问题     | 优化                 |
| -------- | -------- | -------------------- |
| 请求数多 | 没有合并 | 打包、雪碧图、HTTP/2 |
| TTFB 高  | 服务器慢 | 后端优化、CDN        |
| 资源大   | 没压缩   | Gzip、图片压缩       |
| 串行请求 | 依赖链长 | 并行、预加载         |

---

# 六、Application（应用）

**用途**：查看和管理浏览器存储、缓存、Service Worker。

## 6.1 左侧菜单

```
Application
├── Manifest
├── Service Workers
├── Storage
│   ├── Local Storage
│   ├── Session Storage
│   ├── IndexedDB
│   ├── Web SQL
│   └── Cookies
├── Cache
│   ├── Cache Storage
│   └── Application Cache
├── Frames
└── Background Services
```

## 6.2 Local Storage / Session Storage

**区别**：

|          | Local Storage    | Session Storage |
| -------- | ---------------- | --------------- |
| 生命周期 | 永久（除非清除） | 关闭标签页就清  |
| 作用域   | 同源所有标签页   | 当前标签页      |
| 大小     | ~5-10MB          | ~5MB            |

**操作**：直接在面板里增删改查，或 Console 里：

```js
localStorage.setItem('token', 'abc');
localStorage.getItem('token');
localStorage.removeItem('token');
localStorage.clear();
```

## 6.3 Cookies

查看当前域下的所有 Cookie：名称、值、Domain、Path、Expires、HttpOnly、Secure。

**调试**：

- 删除单个 Cookie 测试登出
- 修改 Cookie 测试逻辑
- 看 HttpOnly 是否阻止 JS 访问

## 6.4 IndexedDB

查看结构化数据（如 PWA 缓存、离线数据）。

## 6.5 Cache Storage

查看 Service Worker 缓存的资源。

## 6.6 清除存储

顶部有 "Clear site data" 按钮，一键清空所有存储。

---

# 七、Performance（性能）

**用途**：录制和分析页面运行时性能。

## 7.1 基本流程

1. 点 ⚫ 录制
2. 操作页面（滚动、点击）
3. 停止录制
4. 分析报告

## 7.2 报告结构

**FPS 图表**：帧率，绿色越高越流畅，红色表示卡顿。

**CPU 图表**：CPU 占用，颜色对应不同类型任务。

**火焰图（Main）**：函数调用栈和时间。

**Timing**：关键时间点

```
FP  (First Paint)          首次绘制
FCP (First Contentful Paint) 首次内容绘制
LCP (Largest Contentful Paint) 最大内容绘制
DCL (DOMContentLoaded)     DOM 加载完成
L   (Load)                 页面加载完成
```

## 7.3 常见性能问题

| 现象     | 原因              | 解决                 |
| -------- | ----------------- | -------------------- |
| FPS 掉帧 | 长任务阻塞主线程  | 拆分任务、Web Worker |
| 首屏慢   | 资源大            | 懒加载、代码分割     |
| 内存泄漏 | 定时器/监听器未清 | 手动清理             |
| 频繁重排 | 频繁改 DOM        | 批量更新、虚拟 DOM   |

## 7.4 Memory（内存）面板

- **Heap snapshot**：堆快照，对比两次找泄漏
- **Allocation instrumentation**：实时分配
- **Allocation sampling**：采样分析

---

# 八、Lighthouse（灯塔）

**用途**：一键审计页面质量，给出评分和优化建议。

## 8.1 五大维度

| 维度           | 内容            |
| -------------- | --------------- |
| Performance    | 加载性能        |
| Accessibility  | 无障碍          |
| Best Practices | 最佳实践        |
| SEO            | 搜索引擎优化    |
| PWA            | 渐进式 Web 应用 |

## 8.2 使用

1. 打开 Lighthouse 面板
2. 选设备（Mobile / Desktop）
3. 选类别
4. 点 "Analyze page load"

## 8.3 关键指标

| 指标 | 含义         | 目标    |
| ---- | ------------ | ------- |
| FCP  | 首次内容绘制 | < 1.8s  |
| LCP  | 最大内容绘制 | < 2.5s  |
| TBT  | 总阻塞时间   | < 200ms |
| CLS  | 累积布局偏移 | < 0.1   |
| SI   | 速度指数     | < 3.4s  |

## 8.4 报告解读

- **红色**：必须优化
- **橙色**：建议优化
- **绿色**：良好

每条建议点开有详细说明和修复方法。

---

# 九、其他实用面板

## 9.1 Device Toolbar（设备模拟）

`Ctrl+Shift+M` 打开。

- 模拟手机/平板
- 自定义分辨率
- 模拟网速
- 模拟触摸事件
- 模拟地理位置、陀螺仪

## 9.2 Security（安全）

查看页面的 HTTPS 证书、混合内容警告。

## 9.3 Rendering（渲染）

`⋮` → More tools → Rendering。

实用选项：

- **Paint flashing**：重绘区域闪绿
- **Layout Shift Regions**：布局偏移高亮
- **Frame Rendering Stats**：显示 FPS
- **Emulate CSS media**：模拟 dark mode、打印样式

## 9.4 Layers（图层）

查看页面合成层，调试 `will-change`、`transform` 性能。

## 9.5 Coverage（覆盖率）

查看 CSS/JS 使用率，找出未使用的代码。

```
⋮ → More tools → Coverage → 录制
红色 = 未使用，绿色 = 已使用
```

---

# 十、实战调试场景

## 场景 1：接口返回 500

1. Network → 找那个请求
2. 看 Response，确认错误信息
3. 看 Payload，确认发送参数
4. 看 Headers，确认 token/Content-Type
5. 复制 cURL，在终端重现

## 场景 2：登录后跳转，Token 没带上

1. Network → Preserve log
2. 找跳转后的请求
3. 看 Request Headers，确认有没有 `Authorization`
4. Application → Local Storage，确认 token 存在
5. Sources → 打断点，看代码是否正确读取

## 场景 3：页面卡顿

1. Performance → 录制操作
2. 看 FPS 图表，找红色区域
3. 看 Main 火焰图，找长任务
4. 定位到具体函数，优化

## 场景 4：样式不生效

1. Elements → 选中元素
2. Styles → 看目标样式是否被划掉（被覆盖）
3. Computed → 看最终生效值
4. 检查优先级、`!important`、选择器权重

## 场景 5：内存泄漏

1. Memory → Heap snapshot
2. 操作页面，再拍一张
3. 对比两次，看对象数量增长
4. 定位未释放的引用

## 场景 6：跨域问题

1. Console 看到 CORS 报错
2. Network → 看请求
3. 看 Response Headers 有没有 `Access-Control-Allow-Origin`
4. 看是不是 OPTIONS 预检失败
5. 对照后端配置排查

---

# 十一、高效技巧

## 11.1 快捷键速查

| 快捷键                  | 作用              |
| ----------------------- | ----------------- |
| `Ctrl+Shift+I`        | 打开 DevTools     |
| `Ctrl+Shift+C`        | 选择元素          |
| `Ctrl+Shift+M`        | 设备模拟          |
| `Ctrl+Shift+P`        | 命令面板          |
| `Ctrl+P`              | 快速打开文件      |
| `Ctrl+Shift+F`        | 全局搜索代码      |
| `Ctrl+[` / `Ctrl+]` | 切换面板          |
| `Esc`                 | 显示/隐藏 Console |
| `F8`                  | 继续执行          |
| `F10`                 | 单步跳过          |
| `F11`                 | 单步进入          |
| `Shift+F11`           | 单步跳出          |

## 11.2 命令面板（`Ctrl+Shift+P`）

输入命令快速执行，如：

- `Show Coverage`
- `Capture screenshot`
- `Disable JavaScript`
- `Show Rendering`

## 11.3 全局搜索（`Ctrl+Shift+F`）

在所有源文件中搜索字符串，找代码位置。

## 11.4 常用小技巧

```js
// 1. 快速选中元素
$0                          // 当前选中
$('h1')                     // 第一个 h1
$$('a')                     // 所有 a

// 2. 监听事件
monitorEvents(window, 'click')   // 监听所有 click
unmonitorEvents(window)

// 3. 查看元素上绑定的监听器
getEventListeners($0)

// 4. 复制对象
copy(obj)                   // 复制到剪贴板

// 5. 清空 Console
clear()                     // 或 Ctrl+L

// 6. 查看函数定义
debug(fn)                   // 给函数打断点
undebug(fn)

// 7. 实时监控表达式
monitor(fn)                 // 每次调用打印参数
```

## 11.5 移动端调试

**Android**：

1. 手机开 USB 调试
2. Chrome 访问 `chrome://inspect`
3. 选设备 → Inspect

**iOS**：

1. Safari → 偏好设置 → 高级 → 显示开发菜单
2. 手机设置 → Safari → 高级 → Web 检查器
3. Safari → 开发 → 选设备

---

# 十二、学习路径建议

## 阶段 1：基础（1 周）

- 熟练打开、切换面板
- Elements 改样式、看盒模型
- Console 打印、执行 JS
- Network 看请求、复制 cURL

## 阶段 2：调试（1-2 周）

- Sources 打断点、单步调试
- 条件断点、DOM 断点
- Watch / Call Stack / Scope
- 用 Network 排查接口问题

## 阶段 3：性能（2-3 周）

- Performance 录制分析
- Lighthouse 审计
- Memory 找泄漏
- Coverage 找无用代码

## 阶段 4：进阶（持续）

- 移动端远程调试
- Service Worker 调试
- WebSocket 调试
- 命令面板高级用法

---

# 十三、一句话总结

> DevTools 是 Web 开发的"听诊器"：**Elements 看结构，Console 看日志，Sources 调代码，Network 查请求，Application 管存储，Performance 找瓶颈，Lighthouse 做体检**。每天用，熟能生巧。

---

需要我针对某个面板（比如 Network 或 Performance）做更深入的实战演示吗？或者给你一套**练习任务清单**，边做边学？


# 服务发现（Service Discovery）详解

这是**微服务架构**里的核心概念。我先讲清楚"为什么需要它"，再讲"它是什么"，最后讲"三个工具的区别"。

---

## 一、先理解问题：为什么需要服务发现？

### 场景：单体时代

以前只有一个应用，前端调后端就一个地址：

```
前端 → http://api.myapp.com
```

地址写死在配置文件里，永远不会变。

### 场景：微服务时代

现在拆成了多个服务：

```
订单服务 → 需要调用 用户服务、库存服务、支付服务
用户服务 → 需要调用 通知服务
库存服务 → 需要调用 用户服务
```

**问题来了**：订单服务怎么知道用户服务的地址？

你可能会说："写死在配置里不就行了？"

```js
// 订单服务里
const userServiceUrl = 'http://192.168.1.10:8001';
```

**但这行不通**，因为：

### 服务实例是"动态变化"的

| 变化类型           | 说明                                    |
| ------------------ | --------------------------------------- |
| **扩容**     | 流量涨了，用户服务从 3 个实例变成 10 个 |
| **缩容**     | 流量降了，从 10 个减到 2 个             |
| **故障**     | 某个实例挂了，IP 不能再用               |
| **重启**     | 实例重启后 IP 可能变了（尤其容器环境）  |
| **迁移**     | 服务器下线，实例换到别的机器            |
| **滚动更新** | 部署新版本时，实例逐个替换              |

**在 Kubernetes 里，Pod 的 IP 每次重启都会变**。你写死的地址几分钟后就失效了。

> **核心矛盾**：服务地址是动态的，但调用方需要知道"现在有哪些可用地址"。

**服务发现就是解决这个问题的**。

---

## 二、服务发现是什么？

> **服务发现** = 一个机制，让服务实例**自动注册**自己的地址，让调用方**自动查询**可用地址。

### 两个核心角色

```
┌─────────────────────────────────────────────┐
│              服务注册中心                      │
│         （Service Registry）                  │
│   记录：哪个服务有哪些实例，地址是什么          │
└──────────────┬──────────────────┬────────────┘
               │                  │
        ① 注册 │                  │ ② 查询
               │                  │
        ┌──────▼──────┐    ┌──────▼──────┐
        │  用户服务    │    │  订单服务    │
        │ （提供者）   │    │ （消费者）   │
        └─────────────┘    └─────────────┘
```

### 完整流程

**第一步：服务注册（Provider 上线）**

用户服务启动时，把自己的地址告诉注册中心：

```
用户服务实例 A 启动 → 注册中心：我是 user-service，地址 192.168.1.10:8001
用户服务实例 B 启动 → 注册中心：我是 user-service，地址 192.168.1.11:8001
用户服务实例 C 启动 → 注册中心：我是 user-service，地址 192.168.1.12:8001
```

注册中心现在有一张表：

| 服务名       | 实例地址          | 状态 |
| ------------ | ----------------- | ---- |
| user-service | 192.168.1.10:8001 | 健康 |
| user-service | 192.168.1.11:8001 | 健康 |
| user-service | 192.168.1.12:8001 | 健康 |

**第二步：服务发现（Consumer 调用）**

订单服务要调用户服务，先问注册中心：

```
订单服务 → 注册中心：user-service 有哪些可用实例？
注册中心 → 订单服务：[10:8001, 11:8001, 12:8001]
```

然后订单服务选一个（比如轮询），发起调用：

```
订单服务 → 192.168.1.10:8001/users/1
```

**第三步：健康检查（自动摘除故障实例）**

注册中心定期检查每个实例：

```
注册中心 → 192.168.1.10:8001/health  → 200 OK ✅
注册中心 → 192.168.1.11:8001/health  → 超时  ❌ 标记为不健康，从列表移除
注册中心 → 192.168.1.12:8001/health  → 200 OK ✅
```

下次订单服务查询时，只会拿到 10 和 12。

**第四步：服务下线（优雅关闭）**

实例关闭前主动通知注册中心：

```
用户服务实例 A 即将关闭 → 注册中心：注销我
注册中心：已移除 192.168.1.10:8001
```

---

## 三、两种服务发现模式

### 模式 1：客户端发现（Client-Side Discovery）

调用方自己查注册中心，自己选实例。

```
订单服务 ──① 查询──> 注册中心
订单服务 <──② 返回实例列表── 注册中心
订单服务 ──③ 自己选一个，直接调用──> 用户服务
```

**代表**：Eureka、Consul

**优点**：调用方灵活控制负载均衡策略
**缺点**：每种语言都要实现一套客户端逻辑

---

### 模式 2：服务端发现（Server-Side Discovery）

调用方只请求一个统一入口，由入口负责查和转发。

```
订单服务 ──① 请求──> 负载均衡器/网关
负载均衡器 ──② 查注册中心──> 注册中心
负载均衡器 ──③ 转发──> 用户服务
```

**代表**：Kubernetes Service、AWS ALB

**优点**：调用方简单，不用关心服务发现
**缺点**：多一层，入口可能成为瓶颈

---

## 四、三大工具详解

### 1. Consul（HashiCorp 出品）

**定位**：通用服务发现 + 配置中心 + 健康检查。

**特点**：

- 支持多数据中心
- 内置 KV 存储（可当配置中心）
- 支持 DNS 和 HTTP 两种查询方式
- 有 Web UI
- 语言无关（Go 编写）

**架构**：

```
┌─────────────────────────────────┐
│         Consul Cluster          │
│  ┌────────┐ ┌────────┐ ┌──────┐ │
│  │ Server │ │ Server │ │Server│ │  ← Raft 共识，3或5个
│  └────────┘ └────────┘ └──────┘ │
│       ▲          ▲          ▲   │
│       │          │          │   │
│  ┌────┴───┐ ┌────┴───┐ ┌────┴──┐│
│  │ Agent  │ │ Agent  │ │ Agent ││  ← 每个节点跑一个
│  └────────┘ └────────┘ └───────┘│
└─────────────────────────────────┘
```

**注册方式**：

```json
// 服务配置文件
{
  "service": {
    "name": "user-service",
    "port": 8001,
    "check": {
      "http": "http://localhost:8001/health",
      "interval": "10s"
    }
  }
}
```

**查询方式**：

```bash
# DNS 方式
dig user-service.service.consul

# HTTP 方式
curl http://localhost:8500/v1/catalog/service/user-service
```

---

### 2. Eureka（Netflix 出品）

**定位**：专为微服务设计的服务注册中心，Spring Cloud 生态核心。

**特点**：

- Java 生态，与 Spring Cloud 深度集成
- AP 模型（可用性优先，容忍短暂不一致）
- 客户端发现模式
- 有自我保护机制（防止网络抖动误删实例）

**架构**：

```
┌──────────────┐   ┌──────────────┐
│ Eureka Server│◄──│ Eureka Server│  ← 互相注册，集群
└──────┬───────┘   └──────┬───────┘
       │                  │
  注册 │              注册 │
       │                  │
┌──────▼───────┐   ┌──────▼───────┐
│ 用户服务      │   │ 订单服务      │
│ (Eureka Client)│  │(Eureka Client)│
└──────────────┘   └──────────────┘
```

**Spring Cloud 用法**：

```java
// 提供者：启动时自动注册
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication { ... }

// 消费者：用服务名调用
@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}
```

**注意**：Eureka 2.0 已停止开发，但 1.x 仍在广泛使用。

---

### 3. Kubernetes Service

**定位**：K8s 内置的服务发现，最"云原生"的方案。

**特点**：

- 无需额外部署（K8s 自带）
- 服务端发现模式
- 基于 DNS + kube-proxy
- 与 Pod 生命周期自动绑定

**核心概念**：

```
┌─────────────────────────────────────────┐
│           Kubernetes Cluster            │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   Service: user-service         │    │
│  │   ClusterIP: 10.96.0.10         │    │  ← 稳定虚拟 IP
│  │   DNS: user-service.default.svc │    │  ← 稳定域名
│  └──────────┬──────────────────────┘    │
│             │ 负载均衡                    │
│      ┌──────┼──────┐                    │
│      ▼      ▼      ▼                    │
│   ┌────┐ ┌────┐ ┌────┐                  │
│   │Pod │ │Pod │ │Pod │  ← IP 会变       │
│   └────┘ └────┘ └────┘                  │
└─────────────────────────────────────────┘
```

**定义 Service**：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service      # 匹配带此标签的 Pod
  ports:
    - port: 80
      targetPort: 8001
  type: ClusterIP
```

**调用方式**：

```js
// 直接用服务名当域名
fetch('http://user-service/users/1')
// 或完整域名
fetch('http://user-service.default.svc.cluster.local/users/1')
```

**K8s 自动做的事**：

- Pod 启动 → 自动加入 Service 的 Endpoints
- Pod 挂了 → 自动从 Endpoints 移除
- 新 Pod → 自动加入
- kube-proxy 负责负载均衡

---

## 五、三者对比

| 维度       | Consul        | Eureka            | K8s Service     |
| ---------- | ------------- | ----------------- | --------------- |
| 定位       | 通用服务发现  | Spring Cloud 生态 | K8s 内置        |
| 部署       | 需独立部署    | 需独立部署        | 自带            |
| 语言       | 语言无关      | 主要 Java         | 语言无关        |
| 模式       | 客户端/服务端 | 客户端            | 服务端          |
| 一致性     | CP（Raft）    | AP                | 依赖 etcd（CP） |
| 健康检查   | 主动+被动     | 心跳              | 探针            |
| 配置中心   | ✅ 内置 KV    | ❌                | ConfigMap       |
| 多数据中心 | ✅            | ❌                | 多集群方案      |
| 学习曲线   | 中            | 中（Java 简单）   | 高（要懂 K8s）  |
| 适用       | 混合环境      | Spring Cloud      | 云原生          |

---

## 六、怎么选？

```
你的环境是什么？
│
├── 已经在用 Kubernetes
│   └── 直接用 K8s Service（不用额外引入）
│
├── Spring Cloud 技术栈
│   └── Eureka（生态最顺）
│
├── 多语言、多数据中心、需要配置中心
│   └── Consul
│
└── 小项目、单体
    └── 不需要服务发现（用 Nginx 反代就行）
```

---

## 七、动手体验（Consul 最小示例）

### 1. 启动 Consul

```bash
docker run -d --name consul -p 8500:8500 consul
```

访问 `http://localhost:8500` 看到 Web UI。

### 2. 注册一个服务

```bash
curl -X PUT http://localhost:8500/v1/agent/service/register \
  -d '{
    "Name": "user-service",
    "ID": "user-1",
    "Address": "192.168.1.10",
    "Port": 8001,
    "Check": {
      "HTTP": "http://192.168.1.10:8001/health",
      "Interval": "10s"
    }
  }'
```

### 3. 查询服务

```bash
# HTTP 方式
curl http://localhost:8500/v1/catalog/service/user-service

# DNS 方式
dig @127.0.0.1 -p 8600 user-service.service.consul
```

### 4. 在代码里用

```js
// Node.js 示例：从 Consul 拿实例列表
async function getUserServiceInstances() {
  const res = await fetch('http://localhost:8500/v1/health/service/user-service?passing=true');
  const data = await res.json();
  return data.map(item => `http://${item.Service.Address}:${item.Service.Port}`);
}

// 轮询选一个
const instances = await getUserServiceInstances();
const target = instances[Math.floor(Math.random() * instances.length)];
const user = await fetch(`${target}/users/1`).then(r => r.json());
```

---

## 八、一句话总结

> **服务发现** = 让服务实例自动注册地址、让调用方自动查询地址的机制，解决微服务中"实例动态变化、地址不固定"的问题。Consul 通用、Eureka 偏 Java 生态、K8s Service 云原生自带。

---

## 九、延伸：相关概念

学完服务发现，你还会遇到：

- **API 网关**：统一入口，服务发现的上层
- **负载均衡**：拿到实例列表后怎么选
- **配置中心**：Consul 也能做，还有 Nacos、Apollo
- **服务网格（Service Mesh）**：Istio/Linkerd，把服务发现下沉到 Sidecar
- **Nacos**：阿里开源，服务发现 + 配置中心，国内常用

---

需要我深入讲哪个？比如 **K8s Service 的三种类型（ClusterIP/NodePort/LoadBalancer）**、**服务网格 Istio**，或者 **Nacos 实战**？

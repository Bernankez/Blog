---
title: 埋点
---

# 埋点

埋点是为了获取页面的行为数据，后续再进行统计分析。一般做的埋点监控系统都会涵盖以下三类功能：

1. **用户行为监控**

   统计PV（页面访问次数）、UV（页面访问人数）以及用户点击操作等行为。这些数据可以用来量化工作成果。

2. **页面性能监控**

   一般用来监控页面加载时间、白屏时间等。有了这些性能数据，才能更好的做优化。

3. **错误报警监控**

   收集全局捕获的错误信息，以及代码内被catch的错误告警。获取错误数据，及时处理才能避免大量用户受到影响。

## 分类

- **代码埋点**
  前端直接在代码中埋点
- **全埋点**（自动埋点）
  产品中嵌入SDK，收集用户所有行为数据
- **可视化埋点**
  提供可视化界面，运营直接在网站上可视化操作埋点

## 如何设计实现一个埋点SDK

实现一个SDK，我们需要它具有**多平台**、**可拓展**的特点。

- 为了实现多平台，我们可以每个平台单独一个SDK。我们将核心代码抽出一个core，每个平台继承这个core，实现一些特有的核心方法，比如上报、参数初始化。
- 为了实现可拓展，我们可以将每种监控类型单独抽成一个插件的形式。比如行为监控、性能监控、报警监控都可以单独抽成一个插件，用户可以自行决定使用哪部分功能。（类似高德地图的SDK）

### SDK增加新功能时，如何保证原有业务稳定性

对每个插件、内核方法编写测试用例。在覆盖率达标的情况下，只需保证单元测试通过，就能保证原有业务稳定性。

### SDK的异常隔离与上报

当SDK自身产生异常时，不应影响主业务流程。因此我们可以对每个插件模块单独用try catch包起来。捕获到错误时，进行数据的封装上报。当出现异常后，中止SDK运行并移除所有监听。

### SDK服务端时间校对

如果有一些对时间比较敏感的功能，由于`new Date()`获取到的时间是本地时间，因此我们可以通过获取服务端响应头（Response Header）中的`Date`字段（该字段是服务端发送资源时的服务器时间），计算`Date`与本地时间的差值diff，通过diff校对本地时间。

### 错误上报去重、错误唯一ID

同一用户的同一次会话中，同一个错误只需上报一次即可。可以通过`错误信息`、`错误行号`、`错误列号`、`错误文件`等信息，生成一个hash值，作为错误ID。

### 上报策略

#### 上报方式

- Navigator.sendBeacon（非阻塞，数据量有限制，可以跨域）
- XMLHttpRequest/fetch（sendBeacon的降级）
- Image（GIF、PNG）

#### 上报时机

错误数据、行为数据触发后立即上报

性能数据等待页面加载完成、数据采集完成后上报

#### 上报优化

- HTTP2头部压缩，有效减少请求头大小
- 服务端返回204状态码，省去响应体
- `requestIdleCallback`，浏览器空闲时再发送请求
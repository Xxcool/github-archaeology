# GitHub 考古所 · 编程人格档案

> 输入一个 GitHub 用户名，翻阅你的公开地层，出土一份专属《编程人格档案》。

**在线体验 → [github-archaeology.app.workbuddy.host](https://github-archaeology.app.workbuddy.host/?sharecode=HVE7YZLNGQ)**

不用注册、不用登录，点开就能测，三秒出档案。没有 GitHub 账号也没关系，页内有示例档案可以看完整效果。

![首页](screenshots/01-home.png)

---

## 它挖了些什么

把你的 GitHub 仓库当成一处考古现场，从中出土四样东西：

### 语言年轮

按语言给有推送的仓库分类，画成环形图，占比精确合到 100%。示例账号 18 个仓库：JavaScript 33%、TypeScript 17%、Vue 11%、HTML 11%、Python 6%、其他 22%。偏科还是万金油，一眼看出来。

![示例档案](screenshots/02-archive-demo.png)

### 四维探方

| 维度 | 算法 | 示例值 |
| --- | --- | --- |
| 夜行指数 | 提交时间的小时分布，深夜占比越高分越高 | 92 |
| 命名洁癖 | 命名规范的仓库数 / 仓库总数 | 14/18 → 78 |
| 涉猎广度 | 语言种类数 + 仓库规模 | 9 种语言 / 18 个仓库 → 100 |
| 开源社交 | 关注者与 Star 的量级 | 128 关注 / 268 星 → 85 |

每一项旁边都标了计算依据，不是拍脑袋给分。

### 出土器物

自动挖出四个"文物"：最古老的一层、最重的藏品、最亮的展品、命名化石。

示例账号挖出来的是：最早 7 年前的 `untitled-game`、24.2MB 的 `dm-supply-next`、86 星的 `juejin-csdn-extension`，还有那个一看就是凌晨三点随手敲的 `test`。

### 编程人格 + 判词

四维分数组合成 16 型里的一种，配一句判词。示例账号是「**夜掘·地层编年史官**」：

> 「你把每个深夜都折进仓库，用 commit 给时间做碳十四测年。」

最后可以一键导出 1080×1440 的档案卡，带印章和话题标签，存下来直接发朋友圈。

---

## 技术实现

**单文件 HTML + 原生 JS，零依赖零构建。** 整个应用就是一个 `index.html`，不引任何 CDN、不用打包工具、字体全走系统字体。改成什么重新发布都只是覆盖一个文件。

数据只取 GitHub 三个公开接口，`Promise.all` 并发：

```js
const base = "https://api.github.com";
const [pr, rr, er] = await Promise.all([
  fetch(`${base}/users/${username}`),
  fetch(`${base}/users/${username}/repos?per_page=100&sort=pushed`),
  // events 是可选数据源，失败不该拖垮整个流程
  fetch(`${base}/users/${username}/events/public?per_page=100`).catch(() => null)
]);
```

几个实现取舍：

- **`events` 单独 `catch` 降级**。`/events/public` 对某些账号返回空甚至失败，但它只影响「夜行指数」一项，不该让整个档案挂掉。
- **错误分级**。`404` 是用户不存在，`403/429` 是限流，提示语完全不同。
- **分享卡用 Canvas 2D 导出**。头像需 `crossOrigin="anonymous"`，否则画布被污染、`toBlob` 抛 `SecurityError`——而且 `drawImage` 阶段不报错，报错位置和真因隔了好几行，很容易查错方向。
- **离线兜底**。接口不可用时给出明确原因，并可切到内置示例档案，保证打开的人永远有东西看。

---

## 本地运行

```bash
git clone https://github.com/Xxcool/github-archaeology.git
cd github-archaeology
open index.html      # macOS
# 或者直接双击 index.html
```

不需要起服务，不需要装依赖。

---

## 隐私

只读 GitHub 的公开数据（`/users`、`/users/:name/repos`、`/users/:name/events/public`），不涉及任何授权登录。拿到的数据全部在浏览器本地计算和渲染，**不上传、不落库、不留存**。关掉页面，什么都没留下。

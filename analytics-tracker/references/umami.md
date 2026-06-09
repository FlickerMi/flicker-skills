# Umami 配置指南

## 第一步：基础安装

### 1.1 选择部署方式

**方式 A：Umami Cloud（托管版，免费套餐可用）**

1. 前往 [umami.is](https://umami.is/) → **Sign up**
2. 创建账号后，点击 **Add website**
3. 填写网站名称和域名，点击 **Save**
4. 复制显示的 **Website ID**（格式：UUID，如 `abc12345-xxxx-xxxx-xxxx-xxxxxxxxxxxx`）
5. 复制 **Tracking script URL**（格式：`https://cloud.umami.is/script.js`）

**方式 B：自托管（Docker）**

```bash
docker run -d \
  -e DATABASE_URL=postgresql://user:pass@host/umami \
  -p 3000:3000 \
  ghcr.io/umami-software/umami:postgresql-latest
```

部署后在后台创建网站，获取 Website ID 和脚本地址。

### 1.2 安装追踪脚本

**方式 A：Script 标签（通用）**

在 `<head>` 中添加：

```html
<script
  defer
  src="https://cloud.umami.is/script.js"
  data-website-id="你的-website-id"
></script>
```

**Next.js 项目（`app/layout.tsx`）**：

```tsx
import Script from 'next/script';

<Script
  defer
  src="https://cloud.umami.is/script.js"
  data-website-id="你的-website-id"
  strategy="afterInteractive"
/>
```

**方式 B：npm 包（Next.js / React 推荐）**

```bash
npm install @umami/next
# 或
npm install umami-js
```

```ts
// 无需额外配置，脚本安装后 window.umami 即可用
```

### 1.3 填入 analytics.ts 配置

```ts
const config = {
  primary: 'umami',
  additional: [],
};
```

Umami Website ID 已通过 `data-website-id` 绑定在脚本中，`analytics.ts` 无需再配置 ID。

---

## 第二步：自定义事件配置

Umami 的最大优势：**无需在后台预先配置事件**，代码调用 `window.umami.track()` 后，事件自动出现在报表中。

代码触发事件的方式（已由 `analytics.ts` 封装）：

```ts
// analytics.ts 内部已处理，无需手动调用
window.umami?.track('signup_submit', { method: 'email' });
```

### 可选：为事件添加过滤标签

Umami 后台 → **Events** 页面：
- 可按事件名筛选，查看各事件的触发次数和时间分布
- 无需额外配置，所有 `track()` 调用的事件名自动聚合

---

## 第三步：验证埋点生效

### 3.1 实时面板验证

1. 登录 Umami 后台，进入对应网站的 **Dashboard**
2. 查看右上角的 **Realtime** 视图（显示最近 30 分钟数据）
3. 在浏览器中触发一次埋点操作
4. 刷新 Realtime 视图，应看到新的访问记录

### 3.2 Events 页面验证

1. Umami 后台 → **Events** 标签
2. 选择时间范围为"今天"或"最近 1 小时"
3. 触发操作后等待 10-30 秒，事件应出现在列表中
4. 点击事件名，查看参数（`event data`）是否正确

### 3.3 验证通过标准

- [ ] **Events** 页面出现正确的事件名（如 `signup_submit`）
- [ ] 事件数据（`method: email`）显示正确
- [ ] 多次触发后，计数正确增加

### 3.4 常见问题

**事件没有出现？**
- 检查脚本是否正确加载：浏览器 DevTools → Network → 过滤 `script.js`，确认 200 状态
- 检查 `data-website-id` 是否填写正确
- 检查是否开启了广告拦截插件（Umami 会被部分拦截器误拦）

**本地开发不上报？**
Umami 默认忽略 `localhost`。如需本地测试，在脚本标签添加：
```html
data-domains="localhost,你的域名.com"
```

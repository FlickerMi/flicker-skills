# GA4 配置指南

## 第一步：基础安装

### 1.1 创建 GA4 属性

1. 打开 [Google Analytics](https://analytics.google.com/)
2. 左下角 → **管理** → **创建** → **属性**
3. 填写属性名称（如"我的产品"）、时区、货币
4. 平台选择 **Web**，填写网站 URL 和数据流名称
5. 创建完成后，复制 **衡量 ID**（格式：`G-XXXXXXXXXX`）

### 1.2 安装方式

**方式 A：npm 包（推荐，适合 React/Vue/Svelte 等框架）**

```bash
# 无需安装额外包，直接使用 gtag.js
```

在 HTML 的 `<head>` 中（或框架的 `_document.tsx` / `index.html`）添加：

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

**Next.js 项目（`app/layout.tsx`）**：

```tsx
import Script from 'next/script';

// 在 <head> 中添加：
<Script
  src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"
  strategy="afterInteractive"
/>
<Script id="google-analytics" strategy="afterInteractive">
  {`
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXXXXXXX');
  `}
</Script>
```

**Vue 项目（`index.html`）**：直接在 `<head>` 中添加上述脚本。

### 1.3 填入 analytics.ts 配置

```ts
const config = {
  primary: 'ga4',
  additional: [],
};
```

GA4 无需在代码中再次传入 Measurement ID，脚本安装时已绑定。

---

## 第二步：自定义事件配置

GA4 自动收集 `page_view`、`session_start` 等基础事件，自定义业务事件需在后台注册（可选，但注册后能在报表中更方便地筛选和分析）。

### 2.1 标记自定义事件

1. GA4 后台 → **配置** → **事件**
2. 点击 **创建事件**（可基于现有事件创建，也可直接命名）
3. 输入事件名称（与代码中 `trackEvent()` 的 `name` 参数一致，如 `signup_submit`）
4. 点击 **保存**

### 2.2 标记为转化（关键 P1 事件）

1. GA4 后台 → **配置** → **转化**
2. 点击 **新建转化事件**
3. 输入事件名称（如 `purchase_complete`、`signup_submit`）
4. 保存后，该事件在报表中会显示为转化

> 注意：事件数据通常有 24-48 小时延迟才出现在标准报表，但实时报告和 DebugView 是即时的。

---

## 第三步：验证埋点生效

### 3.1 开启 Debug 模式

在 `analytics.ts` 的分发函数中临时添加 debug 参数：

```ts
if (platforms.includes('ga4')) {
  window.gtag?.('event', name, { ...props, debug_mode: true });
}
```

或在 gtag config 中开启：

```html
gtag('config', 'G-XXXXXXXXXX', { debug_mode: true });
```

### 3.2 使用 DebugView

1. GA4 后台 → **配置** → **DebugView**
2. 在浏览器中触发一次埋点操作（如点击注册按钮）
3. DebugView 页面实时显示事件流，确认事件名称和参数正确

### 3.3 使用实时报告（不需要 debug_mode）

1. GA4 后台 → **报告** → **实时**
2. 触发操作后，**事件计数**区域应出现对应事件名
3. 点击事件名可查看参数详情

### 3.4 验证通过标准

- [ ] DebugView 中出现正确的事件名（如 `signup_submit`）
- [ ] 事件参数（如 `method: 'email'`）显示正确
- [ ] 转化事件在实时转化列表中出现

验证通过后，移除 `debug_mode: true`，保持生产代码干净。

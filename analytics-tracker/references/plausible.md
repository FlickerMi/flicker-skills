# Plausible 配置指南

## 第一步：基础安装

### 1.1 创建 Plausible 账号与网站

**Plausible Cloud（付费，有 30 天试用）**

1. 前往 [plausible.io](https://plausible.io/) → **Start free trial**
2. 注册后，点击 **Add a website**
3. 填写域名（如 `myapp.com`），选择时区，点击 **Add snippet**
4. 记录下页面上显示的**域名**（用于脚本 `data-domain` 属性）

**Plausible Community Edition（自托管，免费）**

```bash
git clone https://github.com/plausible/community-edition
cd community-edition
# 按 README 配置 .env 文件
docker compose up -d
```

### 1.2 安装追踪脚本

**方式 A：Script 标签（通用）**

在 `<head>` 中添加：

```html
<script
  defer
  data-domain="myapp.com"
  src="https://plausible.io/js/script.js"
></script>
```

如需追踪自定义事件（必须），使用带 `tagged-events` 或 `custom-events` 的脚本版本：

```html
<script
  defer
  data-domain="myapp.com"
  src="https://plausible.io/js/script.tagged-events.js"
></script>
```

**Next.js 项目**：

```tsx
import Script from 'next/script';

<Script
  defer
  data-domain="myapp.com"
  src="https://plausible.io/js/script.tagged-events.js"
  strategy="afterInteractive"
/>
```

**方式 B：npm 包（Next.js 推荐）**

```bash
npm install next-plausible
```

```tsx
// app/layout.tsx
import PlausibleProvider from 'next-plausible';

export default function RootLayout({ children }) {
  return (
    <PlausibleProvider domain="myapp.com">
      {children}
    </PlausibleProvider>
  );
}
```

### 1.3 填入 analytics.ts 配置

```ts
const config = {
  primary: 'plausible',
  additional: [],
};
```

与 Umami 类似，Plausible 域名已在脚本的 `data-domain` 中绑定，`analytics.ts` 无需额外 ID。

---

## 第二步：自定义事件（Goals）配置

Plausible 的自定义事件必须**在后台预先声明**，否则事件数据不会显示在报表中。

### 2.1 添加 Goal

1. Plausible 后台 → 网站设置（齿轮图标）→ **Goals**
2. 点击 **Add goal**
3. 选择 **Custom event**
4. 输入事件名称（必须与代码中的 `name` 完全一致，如 `signup_submit`）
5. 点击 **Add goal** 保存

**为每个 P1 和 P2 事件都添加对应的 Goal。**

P1 事件示例：
- `signup_submit`
- `login_success`
- `purchase_complete`

P2 事件示例：
- `plan_select`
- `contact_submit`

### 2.2 查看自定义属性（Props）

Plausible 支持为事件附带属性数据（需在 Goal 设置中开启）：

1. 编辑对应的 Goal → **Custom properties**
2. 添加属性名称（如 `method`、`plan`、`price_usd`）
3. 保存后，属性数据会显示在 Goal 详情页

---

## 第三步：验证埋点生效

### 3.1 实时访客验证

1. Plausible 后台 → 主 Dashboard，点击右上角 **Realtime**
2. 在浏览器中触发埋点操作
3. 实时视图应显示新增访客和对应的目标转化

### 3.2 Goals 页面验证

1. Dashboard → 向下滚动到 **Goal conversions** 区域
2. 或点击顶部 **Goals** 过滤标签
3. 触发操作后（通常 10-60 秒内刷新），对应 Goal 的转化计数应增加

### 3.3 验证通过标准

- [ ] Dashboard 实时视图显示事件触发记录
- [ ] **Goal conversions** 中出现对应事件名
- [ ] 若配置了 Props，属性数据（如 `plan: pro`）正确显示
- [ ] 多次触发后，转化计数正确增加

### 3.4 常见问题

**Goal 触发了但不显示？**
- 确认后台已添加完全匹配的 Goal 名称（大小写敏感）
- 确认使用的是 `script.tagged-events.js` 而非基础 `script.js`

**本地开发不上报？**
Plausible 默认忽略 `localhost`，这是正常行为。如需本地测试，可临时使用 Plausible 提供的浏览器扩展（Plausible Analytics Extension）来绕过过滤。

**使用 next-plausible 包时如何调用自定义事件？**

```ts
// analytics.ts 中的 plausible 分发函数无需修改
// window.plausible 由 next-plausible 自动注入
window.plausible?.('signup_submit', { props: { method: 'email' } });
```

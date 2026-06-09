# analytics.ts / analytics.js 结构模板

## TypeScript 版本（`analytics.ts`）

```ts
// ============================================================
// 区域 1：平台配置
// 修改这里来切换平台，不需要改其他任何地方
// ============================================================
type Platform = 'ga4' | 'umami' | 'plausible';

const config: {
  primary: Platform;
  additional: Platform[];
} = {
  primary: 'ga4',       // 主平台：'ga4' | 'umami' | 'plausible'
  additional: [],       // 可选追加平台，例如 ['umami']
};

// ============================================================
// 区域 2：核心分发函数（样板，通常不需要修改）
// ============================================================
export function trackEvent(
  name: string,
  props?: Record<string, string | number | boolean | undefined>
): void {
  const platforms = [config.primary, ...config.additional];

  if (platforms.includes('ga4')) {
    window.gtag?.('event', name, props);
  }
  if (platforms.includes('umami')) {
    window.umami?.track(name, props);
  }
  if (platforms.includes('plausible')) {
    window.plausible?.(name, { props });
  }
}

// ============================================================
// 区域 3：具名事件函数（由扫描结果生成，每个项目不同）
// 每个函数对应一个业务操作，参数明确传递业务上下文
// ============================================================
export const track = {
  // --- P1 关键事件 ---
  signupSubmit: (method?: 'email' | 'google' | 'github') =>
    trackEvent('signup_submit', { method }),

  loginSuccess: (method?: 'email' | 'google' | 'github') =>
    trackEvent('login_success', { method }),

  purchaseComplete: (plan: string, price_usd: number) =>
    trackEvent('purchase_complete', { plan, price_usd }),

  // --- P2 重要事件 ---
  planSelect: (plan: string) =>
    trackEvent('plan_select', { plan }),

  contactSubmit: () =>
    trackEvent('contact_submit'),

  // --- P3 可选事件（如用户选择追踪）---
  navClick: (label: string) =>
    trackEvent('nav_click', { label }),
};
```

---

## JavaScript 版本（`analytics.js`）

```js
// ============================================================
// 区域 1：平台配置
// ============================================================
const config = {
  primary: 'ga4',     // 'ga4' | 'umami' | 'plausible'
  additional: [],     // 例如 ['umami']
};

// ============================================================
// 区域 2：核心分发函数
// ============================================================
export function trackEvent(name, props) {
  const platforms = [config.primary, ...config.additional];

  if (platforms.includes('ga4')) {
    window.gtag?.('event', name, props);
  }
  if (platforms.includes('umami')) {
    window.umami?.track(name, props);
  }
  if (platforms.includes('plausible')) {
    window.plausible?.(name, { props });
  }
}

// ============================================================
// 区域 3：具名事件函数
// ============================================================
export const track = {
  signupSubmit: (method) =>
    trackEvent('signup_submit', { method }),

  loginSuccess: (method) =>
    trackEvent('login_success', { method }),

  purchaseComplete: (plan, price_usd) =>
    trackEvent('purchase_complete', { plan, price_usd }),

  planSelect: (plan) =>
    trackEvent('plan_select', { plan }),

  contactSubmit: () =>
    trackEvent('contact_submit'),
};
```

---

## 生成规则

生成 `analytics.ts` / `analytics.js` 时：

1. **检测项目语言**：项目根目录有 `tsconfig.json` → 生成 `.ts`，否则生成 `.js`
2. **平台配置区**：按用户选择填入 `primary` 和 `additional`
3. **具名事件函数**：只生成用户在候选清单中确认的事件，P3 默认不生成
4. **参数设计原则**：
   - P1 事件尽量带有业务上下文参数（plan、method、price 等）
   - P2/P3 事件参数可选或无参数
   - 参数类型使用基础类型：`string`、`number`、`boolean`
5. **文件放置位置**：优先 `src/analytics.ts`，若无 `src/` 目录则放在根目录，询问用户确认

---

## 平台类型声明（TypeScript 项目需添加）

若项目无对应类型定义，在 `analytics.ts` 顶部或 `global.d.ts` 添加：

```ts
declare global {
  interface Window {
    gtag?: (command: string, action: string, params?: object) => void;
    umami?: { track: (event: string, data?: object) => void };
    plausible?: (event: string, options?: { props?: object }) => void;
  }
}
```

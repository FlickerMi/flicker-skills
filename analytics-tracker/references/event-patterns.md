# 事件检测规则与优先级分级

## 扫描范围

扫描以下文件类型：`.js` `.ts` `.jsx` `.tsx` `.vue` `.svelte`

跳过以下路径/文件：
- `node_modules/`
- `*.test.*` `*.spec.*`
- `*.config.*` `vite.config.*` `next.config.*`
- `dist/` `build/` `.next/` `.nuxt/`

---

## 事件检测模式

### 1. 点击交互

| 框架 | 检测模式 |
|------|----------|
| React / Next.js | `onClick=` `onClick ={` |
| Vue | `@click=` `v-on:click=` |
| Svelte | `on:click=` `bind:click` |
| 原生 JS | `addEventListener('click'` `.onclick =` |

**关注对象**：`<button>`、`<a>`、含 `btn` / `cta` / `link` class 的元素。

### 2. 表单提交

| 框架 | 检测模式 |
|------|----------|
| React / Next.js | `onSubmit=` `handleSubmit` |
| Vue | `@submit=` `v-on:submit=` |
| Svelte | `on:submit=` |
| React Hook Form | `handleSubmit(` `useForm(` |
| 原生 JS | `addEventListener('submit'` `.onsubmit =` |

### 3. API 调用（业务操作触发点）

检测以下模式（通常紧跟业务逻辑，是 P1 埋点的黄金位置）：

```
fetch(
axios.post(  axios.put(  axios.patch(
useMutation(
$http.post(  $http.put(
trpc.*.mutate(
supabase.from(*.insert(  supabase.from(*.upsert(
```

### 4. 路由跳转

| 框架 | 检测模式 |
|------|----------|
| React Router | `navigate(` `useNavigate(` |
| Next.js | `router.push(` `useRouter(` `<Link href=` |
| Vue Router | `router.push(` `$router.push(` `<router-link` |
| Svelte | `goto(` `<a href=` |

### 5. 业务函数名关键词

函数名（`function`、箭头函数、方法）中含以下词时，自动提升为 P1 候选：

```
signup  register  createAccount
login   signin    authenticate
logout  signout
purchase  checkout  pay  subscribe  upgrade  downgrade
onboard  activate  complete
delete  cancel  remove  unsubscribe
```

---

## 优先级分级规则

### P1 关键（直接影响收入 / 留存）

满足以下任一条件：
- 函数名含业务关键词（见上表）
- API 调用模式 + 路径含 `/auth`、`/payment`、`/subscribe`、`/checkout`、`/order`
- 表单提交 + 组件/文件名含 `signup`、`login`、`checkout`、`payment`

**必须埋点，不可跳过。**

### P2 重要（功能参与度）

满足以下任一条件：
- 表单提交（非 P1）
- API 调用（非 P1 路径）
- 含明显 CTA 文本的按钮：`立即购买`、`免费试用`、`升级`、`开始`、`Get Started`、`Try Free`、`Upgrade`

**建议埋点，可由用户删除。**

### P3 可选（行为数据）

- 普通导航点击
- 通用链接跳转
- 内容类点击（文章、卡片）

**列出但默认不选，用户主动勾选才生成代码。**

---

## 候选清单输出格式

扫描完成后，按以下格式输出候选清单，等待用户确认：

```
已扫描 XX 个文件，发现 XX 个候选埋点位置。

P1 关键（建议全部保留）
  ✦ src/pages/signup.tsx:42       onSubmit → track.signupSubmit()
  ✦ src/components/CheckoutForm.tsx:88  handlePay → track.purchaseComplete()
  ✦ src/api/auth.ts:23            POST /auth/login → track.loginSuccess()

P2 重要（建议保留）
  ◆ src/pages/pricing.tsx:31      onClick → track.planSelect()
  ◆ src/components/ContactForm.tsx:67  onSubmit → track.contactSubmit()

P3 可选（默认不选）
  ○ src/components/Navbar.tsx:15  onClick → track.navClick()
  ○ src/components/BlogCard.tsx:23  onClick → track.articleClick()

请告知：
1. 是否有需要删除的条目？
2. 是否有需要手动添加的位置？
3. P3 中是否有想保留的？
确认后将生成追踪模块。
```

---

## 事件命名规范

- 使用 `snake_case`
- 格式：`{动作}_{对象}`，如 `signup_submit`、`plan_select`、`purchase_complete`
- 避免平台保留字：不使用 `page_view`、`session_start`（这些由平台自动收集）
- 属性名同样使用 `snake_case`，如 `{ plan_name, price_usd }`

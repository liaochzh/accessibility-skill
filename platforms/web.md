# Web/H5 无障碍检查清单

## 检查顺序（不要跳过）

### 1. 语义与结构
- [ ] 「帮助 / 联系」入口在各页保持同一相对顺序（导航或页脚位固定，不随页面漂移）（SC 3.2.6）
- [ ] 交互元素用正确标签：`<button>` 不是 `<div onClick>`，`<a href>` 不是 `<span onClick>`
- [ ] 单页一个 `<h1>`，`h2-h6` 不跳级（h1 → h3 是问题）
- [ ] 列表用 `<ul>/<ol>`，数据表格用 `<table>` + `<th scope>`，不用 div 仿表格
- [ ] 每个 `input` 有对应 `<label>`（`for`/`id` 关联或包裹），占位符不能替代 label
- [ ] 非文本内容有替代：`<img>` 必带 `alt`（有意义→描述信息，不是文件名；装饰→`alt=""`），信息图/图表另给文字等价，图标用 `aria-label` + `aria-hidden`（SC 1.1.1）
- [ ] 地标：`<main>/<nav>/<header>/<footer>` 划分区域

### 2. 键盘与焦点
- [ ] 全部功能可仅用键盘完成：无键盘陷阱，自定义控件绑 Enter/Space 并暴露正确 `role`（SC 2.1.1）
- [ ] 聚焦元素不被 sticky header / 浮层遮挡：`scroll-margin-top` + `scroll-padding-top`（SC 2.4.11）
- [ ] 拖拽类操作（排序、滑块）必须另有按钮或键盘替代路径（SC 2.5.7）
- [ ] 所有交互元素可用 Tab 到达；自定义控件需 `tabIndex={0}`（或 -1 程序控制）
- [ ] DOM 顺序即 Tab 顺序；用 CSS 改变视觉顺序时要同步 DOM 顺序
- [ ] 模态打开时焦点 trap、关闭后回到触发元素；Esc 可关闭
- [ ] 不用 `outline: none` 且无替代；提供 `:focus-visible` 样式，焦点环与背景对比按本仓加严至 3:1（`2.4.13` 未登记，不写号）（SC 2.4.7）
- [ ] 支持 Enter/Space 激活、Esc 关闭、方向键操作列表/菜单

### 3. ARIA 与名称
- [ ] 多步表单不重复索取已提交信息（自动填充 + 回填）；已填值可编辑（SC 3.3.7）
- [ ] 认证允许粘贴与密码管理器，不设记忆题／认知测试（SC 3.3.8）
- [ ] 错误以文本指出字段与原因（非仅颜色/位置）；聚焦错误字段或摘要置顶（SC 3.3.1）（SC 3.3.3）
- [ ] 每个输入有可见 `<label>`；placeholder 不作标签（SC 3.3.2）
- [ ] 高风险提交前可逆/可查/可确认（SC 3.3.4）
- [ ] 图标按钮、纯图片链接有可读文本或 `aria-label` / `aria-labelledby`
- [ ] 状态用 `aria-expanded` / `aria-selected` / `aria-disabled` / `aria-checked`，而非仅靠样式
- [ ] 非语义组件用 `role` 匹配行为（如 `role="button/dialog/tab"`）
- [ ] 自定义控件三要素齐：可访问名称 + 正确 `role` + 当前状态值（`aria-expanded/checked/selected/disabled`）；仅靠 CSS 视觉变化表达状态不算（SC 4.1.2）
- [ ] 动态区域用 `aria-live="polite/assertive"`（toast、错误提示、加载状态）（SC 4.1.3）
- [ ] 表单错误用 `aria-invalid` + `aria-describedby` 指向错误文本

### 4. 视觉与对比
- [ ] 图标按钮点击区 ≥24×24 CSS px（建议 44×44；间距足够时可例外）（SC 2.5.8）
- [ ] 文本与背景对比度 ≥ 4.5:1（大字 3:1）
- [ ] 信息不单靠颜色区分（如红色=错误，需加图标或文字）（SC 1.4.1）
- [ ] 支持 200% 缩放不破坏布局（rem、弹性布局）
- [ ] 尊重系统「减弱动效」：动画/视差/自动播放按 `prefers-reduced-motion` 降级为淡入或直接切换（SC 2.3.3）

## 常见问题 → 修复模板

### div 冒充按钮
```jsx
// 问题
<div onClick={onSubmit}>提交</div>

// 修复
<button type="button" onClick={onSubmit}>提交</button>
```

### 图标按钮无名称
```jsx
<button type="button" aria-label="删除">
  <DeleteIcon aria-hidden="true" />
</button>
```

### 模态焦点管理（React）
```jsx
// 打开时聚焦弹窗内首个元素，关闭后回到触发元素
const triggerRef = useRef(null);
const dialogRef = useRef(null);

useEffect(() => {
  if (open) {
    triggerRef.current = document.activeElement;
    dialogRef.current?.focus();
  } else {
    triggerRef.current?.focus();
  }
}, [open]);

// 弹窗容器：role + aria-modal + Esc 关闭 + 焦点 trap（可用 focus-trap-react）
<div ref={dialogRef} tabIndex={-1} role="dialog" aria-modal="true" aria-label="确认删除">
  …
</div>
```

### 表单错误播报
```jsx
<label htmlFor="email">邮箱</label>
<input
  id="email"
  aria-invalid={!!error}
  aria-describedby={error ? "email-error" : undefined}
/>
{error && <p id="email-error" role="alert">{error}</p>}
```

## 验证建议
- 自动化：axe-core / Lighthouse / WAVE；lint 用 `eslint-plugin-jsx-a11y`
- 命令行：`npx @axe-core/cli <url>`、`npx pa11y <url>`、`npx lighthouse <url> --only-categories=accessibility`
- 手动：全程只用键盘走完核心流程 + NVDA / VoiceOver 实测
- 对比度：用浏览器 DevTools 或 axe 检查

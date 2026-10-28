# Android 无障碍检查清单

## 检查顺序

### 1. 名称与描述
- [ ] 「帮助 / 联系」入口在同一 `TopAppBar` 位，不随页面漂移（SC 3.2.6）
- [ ] 表单状态经 `SavedStateHandle`/`rememberSaveable` 保持，不重复索取已提交信息（SC 3.3.7）
- [ ] 认证接入 Autofill（`autofillHints`），不禁用粘贴（SC 3.3.8）
- [ ] 错误以文本指出字段与原因（非仅颜色/位置）；聚焦错误字段（SC 3.3.1）（SC 3.3.3）
- [ ] 每个输入有可见 label；placeholder 不作 label（SC 3.3.2）
- [ ] 高风险提交前可逆/可查/可确认（SC 3.3.4）
- [ ] 所有交互元素有 `contentDescription`（图标按钮、纯图缺失是 P0）
- [ ] 装饰图片 `contentDescription="@null"` + `importantForAccessibility="no"`，避免噪音；有意义的图片/图标用 `contentDescription` 描述**信息内容**（不是文件名）（SC 1.1.1）
- [ ] 状态变化同步描述（如开关“开/关”），不用仅靠颜色
- [ ] 状态变化可播报：`View.announceForAccessibility()` 或 `accessibilityLiveRegion`，不抢焦点（SC 4.1.3）

### 2. 焦点与 TalkBack 导航
- [ ] 聚焦元素被 AppBar/浮层遮挡时调 `bringIntoView`，并处理 `WindowInsets`/`fitsSystemWindows`（SC 2.4.11）
- [ ] `ItemTouchHelper` 拖拽必须另有菜单按钮替代（SC 2.5.7）
- [ ] TalkBack 焦点顺序符合视觉顺序；必要时用 `accessibilityTraversalBefore/After` 调整
- [ ] 所有交互元素可用 TalkBack 到达并激活（自定义 View 需处理 `performClick` + `isClickable`）
- [ ] 全键盘 / D-pad 可走完全部交互：自定义 View 补 `isFocusable` + `performClick`，无键盘陷阱（SC 2.1.1）
- [ ] 键盘焦点可见：`state_focused` 样式 / Compose `focusRequester` + 焦点指示器，不隐藏系统焦点（SC 2.4.7）
- [ ] 弹窗（Dialog/BottomSheet）弹出时焦点进入，关闭后回到触发元素
- [ ] 相关元素用 `importantForAccessibility` 合并/隐藏，减少滑动噪音

### 3. 自定义控件
- [ ] 自定义 View 覆写 `onInitializeAccessibilityNodeInfo`，设置 `className` / `isCheckable` / `isChecked` 等状态
- [ ] 集合控件（下拉、Tab、列表）正确暴露 role 与选中态
- [ ] 三要素齐：`className`/role + `stateDescription` + 动作标签（`onClickLabel`）；仅靠颜色/图标表达状态不算（SC 4.1.2）

### 4. 视觉与缩放
- [ ] 触控目标 ≥48dp（≈7–10mm，取更严者；勿小于 24×24 CSS px 等效）（SC 2.5.8）
- [ ] 对比度 ≥ 4.5:1（大字 3:1）
- [ ] 信息不单靠颜色区分：错误除红色外另有图标/文字（SC 1.4.1）
- [ ] 支持字体缩放：用 `sp` 做字号，布局用约束/滚动容器，最大字号下不截断不重叠
- [ ] 触摸目标 ≥ 48x48 dp
- [ ] 尊重「移除动画」：用 `ValueAnimator.areAnimatorsEnabled()` / `Settings.Global.ANIMATOR_DURATION_SCALE` 降级动画（SC 2.3.3）

## 常见问题 → 修复模板

### 图标按钮无描述
```kotlin
// Compose
IconButton(onClick = { onDelete() }) {
  Icon(Icons.Default.Delete, contentDescription = "删除")
}
```

```xml
<!-- 传统 View -->
<ImageButton
  android:src="@drawable/ic_delete"
  android:contentDescription="删除"
  android:onClick="onDelete" />
```

### 状态同步（Compose）
```kotlin
Switch(
  checked = enabled,
  onCheckedChange = onChange,
  modifier = Modifier.semantics {
    contentDescription = "推送通知，${if (enabled) "开" else "关"}"
    stateDescription = if (enabled) "开" else "关"
  }
)
```

### 自定义 View 暴露状态
```kotlin
override fun onInitializeAccessibilityNodeInfo(info: AccessibilityNodeInfo) {
  super.onInitializeAccessibilityNodeInfo(info)
  info.className = "android.widget.Switch"
  info.isCheckable = true
  info.isChecked = isChecked
}
```

## 验证建议
- 自动化：Espresso + AccessibilityChecks（`AccessibilityChecks.enable()`）；Compose 用 `createAndroidComposeRule` 断言 `SemanticsNode.contentDescription` / `stateDescription`
- 手动：打开 TalkBack，全流程滑动 + 线性导航 + 按标题导航
- 手动：接外接键盘或开「开关控制」，确认全部功能可达且焦点环可见
- 最大字号 + 粗体下检查布局溢出

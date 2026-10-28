# Flutter 无障碍检查清单

## 检查顺序（不要跳过）

### 1. 语义与结构
- [ ] 「帮助 / 联系」入口在同一 `AppBar` 位或同一导航位，不随页面漂移（SC 3.2.6）
- [ ] 交互元素是否有语义：`Button` 用 `ElevatedButton/TextButton/IconButton`，不用 `GestureDetector` 包 `Container` 冒充按钮
- [ ] 图片是否有语义：有意义图片用 `Semantics(label: ...)` 包裹（label 写**信息内容**，不是文件名），装饰图片用 `ExcludeSemantics` 包裹（SC 1.1.1）
- [ ] 需要合并/拆分语义时是否正确使用 `MergeSemantics` / `Semantics(container: true)`
- [ ] 列表是否可用语义化组件（`ListView` + `ListTile`），读屏能逐项朗读

### 2. 名称与状态
- [ ] 多步表单保持 `TextEditingController`/`FormField` 状态，不重复索取已提交信息（SC 3.3.7）
- [ ] 认证走系统自动填充（`AutofillGroup` + `autofillHints`），不禁用粘贴（SC 3.3.8）
- [ ] 错误以文本指出字段与原因（非仅颜色/位置）；聚焦错误字段（SC 3.3.1）（SC 3.3.3）
- [ ] 每个输入有可见 label；placeholder 不作 label（SC 3.3.2）
- [ ] 高风险提交前可逆/可查/可确认（SC 3.3.4）
- [ ] 图标按钮是否有 `tooltip` 或 `Semantics(label:)`（两者至少其一）
- [ ] 自定义控件是否有 `Semantics(button: true / selected: / enabled:)` 等标志
- [ ] 开关/选中态是否用 `Semantics(toggled: / checked:)` 而非仅靠颜色
- [ ] 三要素齐：`Semantics(label / button / value / enabled / checked)`，状态变化同步语义值（SC 4.1.2）
- [ ] 动态内容（toast、错误提示、加载）是否用 `Semantics(liveRegion: true)` 播报（SC 4.1.3）

### 3. 焦点与键盘
- [ ] 聚焦项被 AppBar/浮层遮挡时调 `Scrollable.ensureVisible`，并留 `SafeArea`/`viewPadding` 余量（SC 2.4.11）
- [ ] `ReorderableListView` 等拖拽交互必须另有上下移按钮或键盘路径（SC 2.5.7）
- [ ] 焦点顺序是否符合视觉顺序；弹窗内是否用 `FocusScope` 限制焦点不外泄
- [ ] 弹窗关闭后焦点是否回到触发元素（保存 `FocusNode`，关闭时 `requestFocus`）
- [ ] 桌面/大屏场景：是否有可见焦点样式（`Focus` widget 包裹，`focused` 态有边框变化），对比按本仓加严至 3:1（`2.4.13` 未登记，不写号）（SC 2.4.7）
- [ ] 自定义快捷键是否通过 `Shortcuts` + `Actions` 实现，而非裸 `RawKeyboardListener`
- [ ] 桌面/键盘场景全部功能可仅用键盘完成：Tab 顺序可控、无键盘陷阱（SC 2.1.1）

### 4. 视觉与缩放
- [ ] 点击/触控区 ≥48dp（≈7–10mm，取更严者；勿小于 24×24 CSS px 等效）（SC 2.5.8）
- [ ] 文字与背景对比度 ≥ 4.5:1（大字 3:1）
- [ ] 信息不单靠颜色传达：错误除红色外另有图标/文字（SC 1.4.1）
- [ ] 是否支持系统字体缩放：不用写死 `fontSize` + 固定高度容器，布局用弹性（`Expanded/Flexible`），`textScaler` 放大后不溢出
- [ ] 触摸目标 ≥ 48x48 dp
- [ ] 尊重系统「减弱动效」：用 `MediaQuery.of(context).disableAnimations` 降级动画/视差（SC 2.3.3）

## 常见问题 → 修复模板

### 图标按钮无名称
```dart
// 问题：读屏只读“按钮”，不知道干什么
IconButton(icon: Icon(Icons.delete), onPressed: onDelete)

// 修复：加 tooltip（自动生成语义 label）
IconButton(
  icon: Icon(Icons.delete),
  tooltip: '删除',
  onPressed: onDelete,
)
```

### GestureDetector 冒充按钮
```dart
// 问题：读屏不识别为按钮，键盘无法激活
GestureDetector(onTap: onSubmit, child: Container(child: Text('提交')))

// 修复：用语义化按钮，或补 Semantics + 焦点
Semantics(
  button: true,
  enabled: true,
  label: '提交订单',
  child: InkWell(onTap: onSubmit, child: Container(child: Text('提交'))),
)
```

### 弹窗焦点管理
```dart
// 打开前保存焦点，关闭后恢复；弹窗内用 FocusScope 包裹
final previousFocus = FocusScope.of(context).focusedChild;

showDialog(
  context: context,
  builder: (_) => Focus(
    autofocus: true,
    child: FocusScope(
      child: AlertDialog(
        title: Text('确认删除'),
        actions: [TextButton(onPressed: () => Navigator.pop(context), child: Text('取消'))],
      ),
    ),
  ),
).then((_) => previousFocus?.requestFocus());
```

### 动态播报
```dart
// 提交失败/成功需要读屏播报
Semantics(
  liveRegion: true,
  label: errorMessage, // 错误文本变化时自动播报
  child: Text(errorMessage),
)
```

## 验证建议
- 自动化：`flutter test` 语义测试（`expect(find.bySemanticsLabel(...), findsOneWidget)`）；`SemanticsTester` 核 `label/value/button/checked`
- 手动：打开 `SemanticsDebugger`（`MaterialApp(showSemanticsDebugger: true)`）看语义树
- 真机：iOS VoiceOver / Android TalkBack 全流程走一遍 + 最大字体下检查溢出

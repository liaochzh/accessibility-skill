# iOS 无障碍检查清单

## 检查顺序

### 1. 名称与值
- [ ] 「帮助 / 联系」入口在同一导航位，不随页面漂移（SC 3.2.6）
- [ ] 表单草稿状态保持，不重复索取已提交信息（SC 3.3.7）
- [ ] 认证支持 `textContentType = .password` 等系统自动填充，不禁用粘贴（SC 3.3.8）
- [ ] 错误以文本指出字段与原因（非仅颜色/位置）；聚焦错误字段（SC 3.3.1）（SC 3.3.3）
- [ ] 每个输入有可见 label；placeholder 不作 label（SC 3.3.2）
- [ ] 高风险提交前可逆/可查/可确认（SC 3.3.4）
- [ ] 非文本内容有替代：有意义的图片/图标用 `accessibilityLabel` 描述**信息内容**（不是文件名/图标名），装饰视图 `isAccessibilityElement = false`（SC 1.1.1）
- [ ] 所有交互元素有 `accessibilityLabel`（图标按钮、纯图片缺失是 P0）
- [ ] 需要补充说明用 `accessibilityHint`（如“双击删除”），不要把 hint 写进 label
- [ ] 有状态的控件同步 `accessibilityValue`（如滑杆当前值、步进器数值）
- [ ] 状态变化可播报：`UIAccessibility.post(notification: .announcement, argument:)`，不抢焦点；**焦点切换**（弹窗 / 整屏）才用 `.screenChanged`（SC 4.1.3）
- [ ] 每个控件三要素齐：`accessibilityLabel` + `accessibilityTraits` + `accessibilityValue`，状态变化同步值（SC 4.1.2）

### 2. 特征（Traits）
- [ ] 按钮用 `.button`，标题用 `.header`，图片按钮组合正确
- [ ] 选中态用 `.selected`，禁用态用 `.notEnabled`，而非仅靠颜色
- [ ] 静态文本不加 `.button`，可点击元素不缺 traits

### 3. 焦点与 VoiceOver 导航
- [ ] 聚焦元素不被导航栏/浮层遮挡：`scrollRectToVisible` + `contentInsetAdjustmentBehavior`（SC 2.4.11）
- [ ] 拖拽交互另有「移动」操作菜单或按钮替代（SC 2.5.7）
- [ ] 外接键盘 / Full Keyboard Access 可走完全部功能：自定义控件补 `keyCommands`，无键盘陷阱（SC 2.1.1）
- [ ] 键盘焦点可见：不隐藏系统焦点环，自定义 Focus 样式与背景对比按本仓加严至 3:1（`2.4.13` 未登记，不写号）（SC 2.4.7）
- [ ] VoiceOver 焦点顺序符合视觉顺序；自定义容器必要时用 `accessibilityElements` 显式排序
- [ ] 弹窗（sheet/alert）弹出时 VoiceOver 焦点进入弹窗，关闭后回到触发元素
- [ ] 装饰视图设置 `isAccessibilityElement = false`，避免噪音焦点
- [ ] 相关元素用 `shouldGroupAccessibilityChildren` 合并，减少滑动次数

### 4. 视觉与动态类型
- [ ] 触控目标 ≥ 44×44pt（勿小于 24×24 CSS px 等效）（SC 2.5.8）
- [ ] 对比度 ≥ 4.5:1（大字 3:1）
- [ ] 信息不单靠颜色区分：错误除红色外另有图标/文字（SC 1.4.1）
- [ ] 支持动态类型：用 `UIFont.preferredFont(forTextStyle:)`，布局用 Auto Layout，最大字号下不截断
- [ ] 尊重「减弱动态效果」：用 `UIAccessibility.isReduceMotionEnabled` / `@Environment(\.accessibilityReduceMotion)` 降级动画（SC 2.3.3）

## 常见问题 → 修复模板

### 图标按钮无名称（SwiftUI / UIKit）
```swift
// SwiftUI
Button(action: onDelete) {
  Image(systemName: "trash")
}
.accessibilityLabel("删除")   // P0 必加
.accessibilityHint("双击删除当前条目")
```

```swift
// UIKit
deleteButton.accessibilityLabel = "删除"
deleteButton.accessibilityHint = "双击删除当前条目"
deleteButton.accessibilityTraits = .button
```

### 状态同步
```swift
// 开关状态要让读屏知道
toggle.isAccessibilityElement = true
toggle.accessibilityLabel = "推送通知"
toggle.accessibilityValue = isOn ? "开" : "关"
toggle.accessibilityTraits = [.button, .selected] // 选中时加 .selected
```

### 弹窗焦点
```swift
// 弹窗出现后播报并聚焦标题
alertView.accessibilityViewIsModal = true // 屏蔽背后元素
UIAccessibility.post(notification: .screenChanged, argument: alertTitle)
```

## 验证建议
- 自动化：XCTest UI 测试断言 `accessibilityLabel` 存在；`XCTAssertTrue(element.isAccessibilityElement)`
- 手动：开 Full Keyboard Access（设置 → 辅助功能 → 键盘）用 Tab 走核心流程，确认焦点环始终可见
- 手动：打开 VoiceOver（设置 → 辅助功能），全流程滑动 + 转子（标题/链接/按钮）导航
- 最大字号（辅助功能 → 显示与文字大小）下检查布局
- 商店标签：用 Xcode → Accessibility Inspector 核对比度与元素属性；每类标签至少一台真机，叠加最大字号 + 深色 + 减弱动效组合走查
- 商店标签结论进报告「商店标签结论」节（标签 / 设备 / 三态 / 取证级别 / 依据项）

## App Store 无障碍营养标签（上线前自检）

> 适用：任何**上架 App Store 的产物**（iPhone / iPad / Mac / Apple TV / Vision Pro / Watch），按**发行平台**判，不按代码框架判——Flutter / React Native 构建的 iOS App 同样走本节。
> 出处：Apple《Overview of Accessibility Nutrition Labels》+ 9 项 evaluation criteria **全量直读**，2026-10-28；只在诉求是「上架 / 商店元数据 / 标签怎么填」时才走本节，日常代码审计用上面 1-4 节。
> 标签在 iOS/iPadOS/macOS/tvOS/visionOS/watchOS 26+ 的产品页展示；App Store Connect 按设备分别填写。

判定总则（任一不满足就不该勾选）：
- **「支持」= 能用该功能完成全部常见任务**（common tasks）——不是能打开首页，也不是部分页面可用；先列 App 的常见任务，再逐项走查。
- 按设备分别评估（同一功能在 iPhone / iPad / Mac / Watch 可结论不同）；**Larger Text 不支持 Mac**，**Voice Control 不支持 Apple TV 与 Apple Watch**。
- **不得用自定义实现替代核心辅助技术**（Apple 明示不要自行重做 VoiceOver / Voice Control 的能力）；也不得拿 Zoom / Hover Text 充当 Larger Text。
- 标签属商店元数据，受 App Review Guideline 2.3 约束，虚标会被要求更正；本 skill 只出「技术上是否达到该标签」，不代填、不出商店合规结论。

| # | 标签 | 支持设备 | 判定口径（一句话） | 常见不通过点 |
|---|---|---|---|---|
| 1 | VoiceOver | 全平台 | 仅用 VoiceOver（手势或键盘）可聚焦并操作全部可见元素，完成全部常见任务 | 元素不可达或漏焦点；前后遍历跳过/循环；模态背后仍可达；自定义控件不等同原生（§1，SC 4.1.2） |
| 2 | Voice Control | iPhone / iPad / Mac / Vision Pro | 仅用语音（Show numbers / Show names + 命名控件）完成点按、手势与文本输入 | 图标按钮无名或名称与可见文字不一致；自定义手势无语音路径；自定义输入框不支持听写 |
| 3 | Larger Text | iPhone / iPad / Apple TV / Vision Pro / Watch | 正文 ≥200% 或系统最大字号下不重叠、不严重截断 | 写死字号 / 固定高度；只放大正文不放大关键图标；拿 Zoom 冒充 |
| 4 | Dark Interface | 全平台 | 默认全流程深色（排除三方 / UGC），或用户可开启深色且保持深色 | 只做浅色模式；灰压黑；Smart Invert 下语义色反转 |
| 5 | Differentiate Without Color Alone | 全平台 | 关键信息不靠颜色单独承载（形状 / 位置 / 顺序 / 图标 / 文字） | 仅红色表错误；仅色点表选中；图表仅靠色相分类（§4，SC 1.4.1） |
| 6 | Sufficient Contrast | 全平台 | 开 Bold Text + Increase Contrast + Reduce Transparency 后仍可读；文本 ≥4.5:1、**非文本与控件状态 ≥3:1**（与 §4「大字 3:1」是两个指标）；深浅色都测 | 只测浅色；灰压黑在增强对比下仍偏低；半透明底上叠文字 |
| 7 | Reduced Motion | 全平台 | 视差 / 缩放 / 旋转 / 多轴 / 自动轮播按系统设置降级，语义不丢 | 只关部分动画；降级后布局塌陷或状态不可感知（§4，SC 2.3.3） |
| 8 | Captions | 全平台 | 视频对白 + 关键音效有时间同步字幕；纯音频可用高质量文字稿 | 仅自动生成未校对；缺音效提示；有功能但库内几乎无字幕内容 |
| 9 | Audio Descriptions | 全平台 | 视频 / 过场 / 关键画面有同步旁白，系统设置开时默认启用 | 只有文字稿；旁白与画面不同步；有功能但库内几乎无 AD 内容 |

结论三态：**可勾选 / 暂不可勾选 / 待真机**，逐项写进报告「商店标签结论」节。

### 逐项细则（Apple 原文要点，2026-10-28 直读）

**1. VoiceOver**
- 所有控件有简洁准确的 label；屏幕上看得见的重要元素，必须能被朗读或盲文感知。
- 每个可见元素都能到达、也能离开；前后遍历与转子遍历都不跳过、不反复循环同一项。
- 多项以组呈现时，要么逐项可达，要么通过自定义动作（回复 / 转发 / 删除）操作整组。
- 后台刷新（如加载更多）不得把阅读位置重置回顶部。
- 整屏切换或模态弹出时焦点逻辑进入新内容；背后不可见 / 离屏内容不得可达。
- 激活手势（选择后双击）与键盘 VO+空格须等价于点按；VoiceOver 开启时标准键盘操作仍可用。
- 上下文菜单、长按等非默认行为用自定义动作暴露（与 VO+Shift+M 等价）；模态须能用 Escape 或 `accessibilityPerformEscape` 关闭。
- 拖放、多指手势要有可发现的替代；自定义控件能力须等同 UIKit / AppKit / SwiftUI 原生控件。

**2. Voice Control**
- 说「Show numbers」「Show names」确认所有按钮 / 链接 / 交互元素都有编号与名称。
- Voice Control 名称必须与界面可见文字一致（不一致会让用户以为功能失效）。
- 滑动、长按、右键要有语音路径（自定义动作或上下文菜单）；hover / 滑动才出现的隐藏 UI 须有纯语音唤出方式。
- 滚动、地图平移、「Zoom out」等多指手势要能说得出。
- 录音、自定义听写可全程用语音开始与停止。
- 任意文本框可「type」输入，可用「Select …」「Delete that」编辑；自定义输入框或输入校验需额外验证。

**3. Larger Text**
- 正文可放大到 ≥200% 或系统最大字号；AX3（>200%）、AX5（正文 >300%）下不重叠、不严重截断。
- 用 Dynamic Type + 自适应布局；自有字号设置的**最大档**不得低于系统最大档，或提供更细的档位定制。
- 不得以 Zoom / Hover Text 充当 Larger Text。

**4. Dark Interface**
- 默认对全部常见任务深色（排除三方与用户生成内容），或提供能保持深色的开关（系统深色 / 自定义深色 / Smart Invert 兜底）。
- 用 Smart Invert 兜底要防语义色反转（红色「删除」翻绿）与媒体画面被反转。
- 半透明视图只要渲染纯深色 UI 即可支持；Increase Contrast / Reduce Transparency 开启时考虑降低透明度。
- 典型失误：只做浅色模式的对比度，忘做深色；灰压黑在 Increase Contrast 下仍可能不可读。

**5. Differentiate Without Color Alone**
- 默认就不靠颜色承载信息（颜色照常用于加速识别，不是删掉颜色）。
- 用「Color Filters → 灰度」走查能否完成全部常见任务（灰度不等于色盲所见，但能暴露问题）。
- 确实只能靠颜色区分时（如红 / 绿状态点），必须补形状 / 位置 / 顺序 / 图标差异，或提供多种配色方案。
- 图表：颜色顺序与图例一致，或直接在图内标注。

**6. Sufficient Contrast**
- 测前开 Bold Text + Increase Contrast + Reduce Transparency；文本 ≥4.5:1、非文本与控件状态 ≥3:1；深浅色同测；半透明材质上文字可读。

**7. Reduced Motion**
- 深度模拟（视差、动态模糊、景深）、多轴 / 多速 / 旋转 / 漩涡、自动轮播：系统开启减弱动效时停用或替换。
- 纯装饰动画可直接停；承载语义的动画（状态变更、层级切换）不得直接删，改用溶解、高亮渐变、颜色变化等无位移替代。
- App 本身无问题动效 ⇒ 可直接标支持。

**8. Captions**
- 视频对白必须有字幕（主语言与备用语言）；升级到闭路字幕 / SDH 时补非语音音效（如 [门铃响]）。
- 游戏过场、插播等出现对白 / 关键声的位置同样要字幕，不限于标准播放器。
- 纯音频（歌词、播客）可用高质量文字稿标支持，文字稿不要求时间同步（watchOS 亦适用）。
- 三方媒体：须让用户识别哪些内容有字幕；功能在但库内几乎没有字幕内容 ⇒ 不得标支持。

**9. Audio Descriptions**
- 视频须有与画面同步的旁白，覆盖无对白的关键画面（开场、过场、提示、只能靠视觉获得的信息）。
- 系统辅助功能设置开启时默认启用 AD 音轨。
- 三方媒体：须让用户识别哪些内容有 AD；库内几乎没有 AD 内容 ⇒ 不得标支持。

# accessibility

一个用于 **无障碍体验优化** 的 agent skill：审代码时是检查清单，写代码时是修复模板。覆盖 Flutter / iOS / Android / Web-H5 四个平台，按「谁受影响 → 哪类辅助技术 → 取证级别」出结论。

> 面向 opencode / Claude 一类支持 `SKILL.md` 的 agent 运行时。纯提示词实现，无依赖、无脚本、不联网也能用。

## 它解决什么问题

| 常见失败 | 本 skill 的做法 |
|---|---|
| 「加了 label 就算无障碍了」 | 三级取证（静态代码 / 自动化 / 真机）+ 明确「三级不可合并成『无障碍已通过』」 |
| 上来就列缺陷，说不清影响谁 | 步骤2 先定**受影响用户与辅助技术**，说不出受影响用户的「问题」一律降 P3 或不报 |
| 报告缺口乱编号，甚至编造 WCAG 条款 | 准则号只能用上游基线已登记的号，否则写「平台最佳实践」 |
| 静态扫描通过就宣布合规 | **不做合规结论**：静态 + 自动化 + 读代码都不等于符合 WCAG；法务口径只出差距清单 |
| 修完没复验 | 步骤5 重跑同一条清单项 + 补真机项，返工最多 2 轮，结案写「已复验 / 未复验 / 残留风险」 |
| 不知道商店那 9 项标签能不能勾 | iOS 清单含「App Store 无障碍营养标签」节：9 项 × 支持设备 × 判定口径 × 常见不通过点 + 逐项细则 |

## 何时触发

用户说「无障碍 / 可访问性 / a11y / 键盘导航 / 读屏适配 / WCAG / 焦点管理」，提到 VoiceOver / TalkBack / Semantics / 仅用键盘操作 / 视障，提供 axe / Lighthouse / WAVE 报告要修复，或问「App Store 无障碍标签 / Nutrition Labels / 上线前无障碍自检」时。

不负责：通用视觉设计（用 frontend-design / app-design）、代码正确性审查（用 review）。

## 安装

```bash
# 方式一：整个目录放到 skills 根目录
git clone https://github.com/liaochzh/accessibility-skill.git
cp -R accessibility-skill ~/.config/opencode/skills/accessibility

# 方式二：软链（便于跟随仓库更新）
ln -s "$(pwd)/accessibility-skill" ~/.config/opencode/skills/accessibility
```

不同运行时的 skills 目录不同（例如 Claude Code 的 `~/.claude/skills/`），把本目录放进去即可，`SKILL.md` 的 frontmatter（`name` / `description`）就是触发依据。

## 结构

```
SKILL.md                 # 主体：触发条件 → 步骤0-6 → 优先级 → 输出模板 → 输出纪律 → 维护说明
platforms/flutter.md     # Semantics 语义树 / FocusNode / 语义树测试
platforms/ios.md         # UIAccessibility / VoiceOver / 动态类型 + App Store 营养标签（9 项）
platforms/android.md     # TalkBack / contentDescription / 字体缩放
platforms/web.md         # 语义化 / ARIA / 键盘导航 / 对比度
docs/                    # 本地 plan / review（.gitignore 不发布）
```

## 设计要点

- **取证分级**：静态代码证据只能证明「代码里缺」，读屏行为必须真机验证；报告里每条结论都带取证级别。
- **受影响用户优先**：先写清挡住的是读屏、仅键盘、低视力、减弱动效还是语音控制用户，再判是不是问题。
- **不合并成「通过」**：静态项消失 ≠ 问题消失；涉及朗读、焦点顺序、大字号重排的必须补真机项。
- **商店标签只出技术判定**：`platforms/ios.md` 的 App Store 节按 Apple 各功能的 evaluation criteria 写（「支持」= 能用该功能完成全部常见任务，按设备分别评估），结论三态「可勾选 / 暂不可勾选 / 待真机」，不代填、不出商店合规结论。
- **推断标 `[待核]`**：推断的标签文案、属性取值不写成事实。

## 与 design-spec 的关系

WCAG 条款号（`（SC x.y.z）`）的**唯一登记处**是 upstream 基线：`liaochzh/design-spec` 的 `references/ui-design-philosophy.md` §无障碍基线；本仓库只引用不重述。只装本仓库时，清单条目照常可读可用；若要用上游校验器（`design-spec/scripts/check-design-spec.py`）核对「未登记号 / 逐平台落点」，需同时取得 design-spec。

## 关于效果验证

本 skill 的判据来自 WCAG 2.2 AA 与 Apple / Material 官方文档的直读（具体日期见 `SKILL.md` 维护说明），但**没有做过对照实验**证明它能提高审计质量。它的作用是让结论可复现、可反驳：每条都有取证级别、受影响用户与验证方式，便于你逐条核。

## License

MIT（见 `LICENSE`）。

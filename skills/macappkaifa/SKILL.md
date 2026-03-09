---
name: macappkaifa
description: |
  macOS 原生应用开发工作流，涵盖 Swift/SwiftUI 菜单栏应用、状态管理、事件触发、WidgetKit 通知中心小组件与 App Group 数据共享。
  触发场景：用户要做 Mac 应用、菜单栏应用、状态栏应用、SwiftUI、Widget 小组件、MenuBarExtra、通知中心组件、macOS 原生开发、事件触发、状态管理、前端 UI。
  关键词：Mac 应用、菜单栏、状态栏、Swift、SwiftUI、WidgetKit、MenuBarExtra、App Group、@Observable、角标、Popover、Template 图标、Liquid Glass、透明玻璃、Apple 设计。
---

# Mac 应用开发

## 角色定位

你是熟悉 macOS 原生开发生态的开发工程师，严格遵循 Apple 文档与当前最佳实践，注重**事件触发**、**状态管理**与**前端 UI** 的一致性，并采用 **Apple 设计风格**（含透明玻璃 / Liquid Glass）。

## 核心原则（必须遵守）

1. **事件驱动**：菜单栏点击、Popover 内添加/勾选/删除、Widget 点击跳转等，所有交互都有明确事件源与响应路径，不依赖隐式行为
2. **单一数据源**：使用 `@Observable`（或 ObservableObject）作为唯一状态源，持久化通过 App Group 与主 App、Widget 共享，避免多份数据不同步
3. **UI 与系统一致**：菜单栏使用 Template Image（单色、透明背景），Popover 使用 SwiftUI 系统控件（List、Button、TextField），深色/浅色随系统
4. **设计风格**：采用 Apple 设计语言；macOS 26+ 优先使用 Liquid Glass（`.glassEffect()`、`.buttonStyle(.glass)` 等），透明玻璃效果与系统材料一致；检索互联网获取 Apple HIG 与 Liquid Glass 最佳实践
5. **每个项目建立 thesis**：每个 Mac 应用项目必须在仓库内建立一份 **THESIS.md**（建议放在 `docs/` 下，或项目约定的设计规范路径），作为该项目的设计 thesis，写明：设计语言（Apple / Liquid Glass）、色彩与材料、事件/状态/UI 约定、与 HIG 的对应关系；后续开发与迭代以 thesis 为统一指导，不偏离
6. **精确定位**：不依赖记忆，用 grep 精确定位相关类型与文件
7. **最小化修改**：不修改已有 skills；若项目内已有代码，尽量复用、少动
8. **中文回复**：始终使用中文回复
9. **检索最佳实践**：涉及新 API、系统版本或设计规范时，检索互联网确认 MenuBarExtra、WidgetKit、Liquid Glass、Apple HIG 等用法

## 与其他 Skill 的配合

| 场景 | 使用 Skill | 说明 |
|------|------------|------|
| 编译/构建检查 | daimagoujian | 仅适用于 TypeScript/Node 项目；Mac 项目用 Xcode 构建或 `swift build` |
| Bug 修复 | bugxiufu | 问题定位、最小化修复、验证 |
| 代码审查 | daimashencha | 提交前检查、逻辑与一致性 |
| Git 操作 | gitcaozuo | 提交、合并、推送前按需使用 |
| 多任务拆解与并行 | agentdiaodu | 复杂需求拆成多 Agent 时使用 |

## 开发流程

### 第一步：需求与形态确认

1. 明确应用形态：仅菜单栏、菜单栏 + 主窗口、菜单栏 + Widget 等
2. 明确事件：菜单栏点击展开什么、Popover 内有哪些操作、Widget 是否可点击跳转
3. 明确状态：哪些数据需要持久化、是否与 Widget 共享（需 App Group）

### 第二步：项目结构（脚手架）

- 主 App Target：SwiftUI App 生命周期，入口用 `@main`，场景使用 `MenuBarExtra`
- 若需通知中心小组件：新建 Widget Extension Target，勾选 macOS；主 App 与 Widget 启用同一 App Group
- 资源：菜单栏图标使用 Template Image（16×16 @1x、32×32 @2x），命名含 `Template`
- 仅菜单栏不显示 Dock：Info.plist 中 `LSUIElement` = true

### 第三步：状态管理

1. **模型**：定义数据结构（如 TodoItem：id, title, isCompleted, createdAt），Codable 便于持久化
2. **Store**：使用 `@Observable`（iOS 17+ / macOS 14+）或 `ObservableObject`，持有数组与派生属性（如未完成数量）
3. **持久化**：`UserDefaults(suiteName: "group.xxx")` 或 App Group 容器内 JSON 文件；每次增删改后写入，启动与 Widget 读取同一 suite
4. **角标**：菜单栏未完成数量由 Store 派生，在 `MenuBarExtra` 的 label 中显示（如 `Text("\(count)")` 当 count > 0）

### 第四步：事件触发

1. **菜单栏**：`MenuBarExtra` 点击由系统触发展开/收起；内容用 SwiftUI 视图（复杂布局用 `.menuBarExtraStyle(.window)`）
2. **Popover 内**：添加（按钮/Enter）→ 写入 Store 并持久化；勾选/删除→ 更新 Store 并持久化；写成功后调用 `WidgetCenter.shared.reloadAllTimelines()` 刷新 Widget
3. **Widget**：`widgetURL` 或 Link 跳转主应用（URL scheme 或打开主 App）

### 第五步：前端 UI 与设计风格

1. **先读项目 thesis**：开发或改 UI 前先阅读项目内 **docs/THESIS.md**（或约定的设计规范路径），按其中约定的 Apple 风格、透明玻璃与材料、色彩与布局执行
2. **菜单栏**：Template 图标 + 可选数字角标；风格与系统其他图标一致
3. **Popover**：标题、列表（List）、每行勾选+删除、底部或顶部输入+添加；使用系统样式；macOS 26+ 可对容器或按钮使用 `.glassEffect(in:)`、`.buttonStyle(.glass)` 等，实现透明玻璃与系统一致
4. **Widget**：小/中尺寸 SwiftUI 视图，只读展示；可点击打开主应用；若系统支持，Widget 背景可采用与 thesis 一致的材料或玻璃效果

### 第六步：验证

1. 菜单栏点击展开/收起正常
2. 添加、勾选、删除后列表与角标立即更新，重启 App 后数据仍在
3. Widget 显示与主 App 一致，写数据后 Widget 能刷新（或下次刷新时一致）

## 关键检查项

- [ ] 项目已建立 THESIS.md（如 docs/THESIS.md），并作为 UI 与设计决策的指导
- [ ] 所有用户操作都有明确事件路径（事件触发）
- [ ] 单一数据源 + App Group 持久化，主 App 与 Widget 读同一份数据（状态管理）
- [ ] 菜单栏图标为 Template Image；Popover/Widget 符合 thesis 与 Apple 风格（含透明玻璃/Liquid Glass 若适用）（前端 UI）
- [ ] 未修改已有 skills；如需改则最小化
- [ ] 新 API、系统版本与设计规范已检索最佳实践

## 参考技术点

- **MenuBarExtra**：`menu` 风格为下拉列表，`window` 风格为 Popover 式窗口；macOS 13+
- **@Observable**：Swift 5.9+，替代 ObservableObject；macOS 26 起 AppKit/UIKit 对 Observable 有更好支持
- **App Group**：Capability 中勾选，suiteName 一致；用于主 App 与 Widget 共享 UserDefaults 或文件
- **WidgetKit**：TimelineProvider 从 App Group 读数据；主 App 写完后可调用 `WidgetCenter.shared.reloadAllTimelines()`
- **Liquid Glass（macOS 26+）**：`.glassEffect()`、`.glassEffect(in: .rect(cornerRadius:))` 用于视图；`.buttonStyle(.glass)` 用于按钮；避免在 Menu 的 label 内嵌套带 glass 的 Button 导致关闭时残影，应对 Menu 整体使用 `.buttonStyle(.glass)`；旧系统可用 `.ultraThinMaterial` 等材料或自定义描边/阴影模拟玻璃
- **Apple HIG**：Human Interface Guidelines — 设计语言、材料、动效与布局需与官方文档一致
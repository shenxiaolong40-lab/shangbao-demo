# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介

投保链路演示项目（shangbao-demo），模拟美团保×众安保险「住院保·百万医疗（免健告）」的完整投保交互链路。

- GitHub: https://github.com/shenxiaolong40-lab/shangbao-demo（公开）
- 本地路径: `/Users/shenxiaolong/learning/shangbao`

## 技术栈

单文件纯前端项目（无构建工具，无依赖）：
- 唯一文件：`index.html`，包含内联 CSS + JavaScript
- 手机壳模拟器：375×780px，固定宽高，四周圆角，overflow:hidden
- 无框架，原生 DOM 操作 + requestAnimationFrame 驱动动画

## 页面结构

```
.phone
├── .page-content（可滚动主内容，点击开启保障后 blur）
│   ├── .status-bar       状态栏（9:41 / 信号 / WiFi / 电池）
│   ├── .nav-bar          导航栏（返回 + 标题 + 图标）
│   ├── .hero             英雄区（品牌 + 标题 + 副标题）
│   ├── .promo-banner     美团精选 banner（30天免费体验提示）
│   ├── .card             主保障额度卡片（600万 + 权益两列）
│   ├── .card-protection  保障详情卡片（明细 + 免费体验 banner）
│   └── .footer-links     底部协议链接
├── .cta-bottom           悬浮底部按钮区
│   └── #startBtn         「开启保障」按钮（含「30天免费」角标）
├── #mask                 遮罩层
├── #popup                激活弹窗（深金曜石风格）
├── #abandonBtn           放弃保障权益按钮（支付面板出现后显示）
└── #paySheet             支付密码面板（底部弹起）
```

## 交互流程

### 1. 激活流程 `startActivation()`
触发：点击「开启保障」按钮
- 页面内容 blur（`.page-content.blurred`）
- 遮罩渐显（`.mask.show`）
- 150ms 后弹窗入场（`.popup.show`，animtion: popupEntrance）
- 300ms 后启动 requestAnimationFrame 进度动画（tick）

### 2. 进度动画 `tick(ts)` — TOTAL_MS = 4500ms
| 时间点 | 事件 |
|--------|------|
| 200ms  | 第1条权益行滑入（💰现金奖励） |
| 700ms  | 第1条「已激活」badge 弹出 |
| 1200ms | 第2条权益行滑入（🎁免费体验） |
| 1700ms | 第2条 badge 弹出 |
| 2200ms | 第3条权益行滑入（🏆全年保障） |
| 2700ms | 第3条 badge 弹出 |
| 3200ms | 第4条权益行滑入（🔄全额退） |
| 4000ms | 第4条 badge 弹出 |
| ~900ms | 盾牌勾 (#shieldCheck) 渐显 |
| 4500ms | 触发 onComplete() |

### 3. 完成态 `onComplete()`
- 进度条填满 100%
- 成功光晕显示（`#successOverlay.show`）
- 状态文字变为「所有权益已锁定」
- 「放弃保障权益」按钮隐藏
- 1000ms 后：弹窗淡出（`.hide-for-pay`）、遮罩加深（`.dim-for-pay`）、支付面板上滑（`#paySheet.show`）、放弃按钮出现（`#abandonBtn.show`）

### 4. 支付密码交互
- `pressPayKey(num)`：填入数字，更新圆点填充（`.pay-dot.filled`）
- `delPayKey()`：退格
- 输入满 6 位 → 180ms 后 `showPaySuccess()`：隐藏键盘，显示「保障已激活」成功态
- `closePaySheet()`：关闭面板，400ms 后 `resetPaySheet()` 重置状态

### 5. 取消流程 `cancelActivation()`
触发：「放弃保障权益」按钮（两处：弹窗内 #cancelBtn、面板层 #abandonBtn）
- 仅在 `done === false` 时有效（已完成后不可取消）
- 取消 RAF + interval，弹窗/遮罩淡出缩放退场
- 260ms 后全量重置：进度条归零、4 条权益行重置、盾牌勾隐藏、状态文字还原、按钮重新可点

## 关键 CSS 设计

### 激活弹窗（深金曜石风格）
- 背景：多层深蓝黑渐变（`.popup`）
- 顶部光带：黄金线性渐变（`::before`）
- 右上角琥珀光晕 + 左下角红色辉光（`::after` + `.popup-corner-glow`）
- 扫光动效：`shimmerSweep` 每 3.5s 一次（`.popup-shimmer::after`）
- 进度条：金橙渐变 + 霓虹发光 + 右端粒子光点（`::after`）

### 权益行激活态
- 初始：`opacity:0; transform:translateX(-14px)`
- `.visible`：滑入显示
- `.sweep`：金色左边框加深 + 微光背景

### 支付面板
- `transform:translateY(100%)` → `.show` 时 `translateY(0)`，过渡使用 `cubic-bezier(0.32,1.1,0.4,1)`
- 圆点填充：`.pay-dot.filled`（背景变深灰 + 边框）

## 状态变量（JavaScript 全局）

| 变量 | 说明 |
|------|------|
| `TOTAL_MS = 4500` | 激活动画总时长 |
| `startTime` | RAF 起始时间戳 |
| `rafId` | requestAnimationFrame ID，用于 cancel |
| `done` | 是否已完成激活，防止取消 |
| `rowAt[4]` | 4 条权益行出现时间点 |
| `badgeAt[4]` | 4 个 badge 出现时间点 |
| `rowDone[4]` / `badgeDone[4]` | 各行/badge 是否已触发 |
| `shieldDone` | 盾牌勾是否已显示 |
| `payInput[]` | 支付密码输入数组（最多6位） |

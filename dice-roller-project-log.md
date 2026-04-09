# 今天过DUE了吗？ - 项目记录

> 创建日期: 2026-04-08
> 最后更新: 2026-04-09
> 状态: 已部署，持续迭代
> 线上地址: https://jevons0825.github.io/dice-roller/
> 仓库地址: https://github.com/jevonS0825/dice-roller

---

## 项目背景

微信群里5人小组每次分题目会有冲突（多人选同一题）。做了一个网页工具：大家打开链接，各自掷骰子，系统按点数高低自动分配题目。

---

## 技术栈

| 组件 | 选择 | 说明 |
|------|------|------|
| 前端 | 单个 `index.html`，vanilla JS + CSS | 无构建工具，无框架 |
| 实时同步 | Firebase Realtime Database (Spark 免费计划) | 支持多人实时掷骰 |
| QR码 | QRCode.js (CDN) | 方便微信扫码分享 |
| 部署 | GitHub Pages | 免费静态托管 |

全部免费，无需付费。

---

## 用户流程

1. 一人打开网页，创建房间（输入题目列表、参与人数）
2. 生成房间号 + QR码/链接，分享到微信群
3. 其他人打开链接，输入昵称加入
4. 每人点击掷骰子（3D骰子动画），结果实时同步
5. 全员掷完后，按点数高低自动分配题目，显示结果

---

## 文件结构

```
Google Drive/Spark/dice-roller/
├── index.html                  # 整个应用（HTML + CSS + JS，约1100行）
└── dice-roller-project-log.md  # 项目记录
```

---

## Firebase 配置

- 项目名: `dice-roller`
- 项目ID: `dice-roller-e3d7e`
- Database URL: `https://dice-roller-e3d7e-default-rtdb.firebaseio.com`
- GitHub 账号: `jevonS0825`（原 `js3892edu`，已改名）

### Firebase 数据模型

```json
{
  "rooms": {
    "ABC123": {
      "meta": {
        "name": "房间名",
        "createdAt": timestamp,
        "status": "waiting | rolling | done",
        "maxPlayers": 5
      },
      "topics": { "0": "题目1", "1": "题目2", ... },
      "participants": {
        "player_id": {
          "name": "昵称",
          "joinedAt": timestamp,
          "roll": 73,
          "rolledAt": timestamp
        }
      },
      "assignments": {
        "player_id": [0, 2]
      }
    }
  }
}
```

### 安全规则（已设置）

```json
{
  "rules": {
    "rooms": {
      "$roomId": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

---

## 分配算法

1. 按骰子点数降序排列（平局按掷骰时间先后排）
2. 题目数 = 人数 → 一人一题
3. 题目数 > 人数 → 均分，点数高的多分
4. 题目数 < 人数 → 点数低的人本轮无题

---

## 已修复的 Bug

### Bug 1: 骰子点数显示 "undefined"（2026-04-08）

**现象**: 所有参与者的骰子点数显示为 "undefined"
**原因**: Firebase `transaction()` 中使用 `ServerValue.TIMESTAMP` 导致写入不可靠，roll 值未成功写入数据库
**修复**: 将 `transaction()` 改为 `update()`，`ServerValue.TIMESTAMP` 改为 `Date.now()`

### Bug 2: 所有题目分配给一个人（2026-04-08）

**现象**: 分题结果中只有一个人获得了所有题目
**原因**: `checkAllRolled` 用 `p.roll !== null` 判断，当 roll 为 `undefined`（未成功写入）时也被当成"已掷"，导致提前触发分配，此时只有部分参与者数据完整
**修复**: 所有 roll 检查改为 `typeof p.roll === 'number'` 严格判断

---

## 设置过程记录

### Firebase 设置步骤

1. 去 console.firebase.google.com 创建项目 `dice-roller`
2. 左侧 Build → Realtime Database → Create Database
3. 位置选 us-central1，安全规则选锁定模式
4. Rules 标签中替换为上述安全规则，点 Publish
5. Project Overview → Web 图标 (</>) → 注册应用 → 获取 firebaseConfig
6. 配置已填入 index.html

### GitHub Pages 部署步骤

1. 安装 GitHub CLI: `brew install gh`
2. 登录: `gh auth login`（选 GitHub.com → HTTPS → Login with web browser）
3. 在 dice-roller 目录: `git init && git add . && git commit -m "init"`
4. 创建仓库并推送: `gh repo create dice-roller --public --source=. --push`
5. 启用 GitHub Pages: 通过 API 设置 source 为 main 分支

### GitHub 用户名变更

- 原用户名: `js3892edu`
- 新用户名: `jevonS0825`
- 变更后需重新 `gh auth login`
- 所有仓库 URL 自动更新

---

## 待验证

- [ ] 多人实际掷骰子后点数正确显示
- [ ] 全员掷完后题目正确分配给每个人
- [ ] 刷新页面后能重连（localStorage playerId）
- [ ] 微信内置浏览器兼容性
- [ ] QR码扫码能正常打开

---

## 版本记录

### v3 - Apple 风格 UI 重设计（2026-04-09，Gemini CLI）
- 主色调从紫色改为红色（#B31B1B），Apple 设计语言
- SF Pro 字体栈，更扁平简洁的视觉风格
- 头像颜色改为 Apple 系统色

### v2 - UI 全面美化 + 改名（2026-04-09，Claude Code）
- 改名：骰子分题器 → 今天过DUE了吗？
- 毛玻璃卡片、渐变背景、3D骰子放大至140px
- 掷骰按钮呼吸脉冲动画、进度条流光效果

### v1 - 初始版本（2026-04-08）
- 基础功能：创建房间、加入、3D骰子掷骰、自动分题
- Firebase 实时同步、预分配唯一数字消除冲突

---

## 后续可改进

- 增加"重新开始"按钮（不用每次创建新房间）
- 加入掷骰子音效
- 支持房间历史记录
- 加强安全规则（防止恶意篡改他人数据）
- 考虑添加房间密码功能

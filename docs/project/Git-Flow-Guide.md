# Git Flow 开发流程指南

## 1. 项目概述

本项目为月球熔岩洞机器人智能自主采样任务，采用 Git Flow 模型结合 GitHub Projects (V2) 进行项目管理，适用于 4-10 人团队的 C/C++（嵌入式）+ Python（AI）混合技术栈项目。

### 技术栈

- **代码托管**: GitHub
- **项目管理**: GitHub Projects (V2) - Sprint 看板
- **CI/CD**: GitHub Actions
- **嵌入式**: C/C++（编译检查、静态分析、单元测试）
- **AI/仿真**: Python（代码格式、lint、单元测试）

### 分支结构概览

```
main          ← 永久分支，生产环境代码（保护：PR + 2人 review）
  ↑
develop       ← 永久分支，开发集成分支（保护：PR + 1人 review）
  ↑
  ├── feature/123-xxx  ← 功能分支（从 develop 创建）
  └── feature/124-yyy  ← 功能分支（从 develop 创建）

hotfix/x.x.x ← 热修复分支（从 main 创建，合并到 main + develop）
```

---

## 2. 分支命名规范

| 分支类型 | 命名规范                        | 生命周期 | 保护规则               |
| ------- | --------------------------- | ---- | ------------------ |
| main    | `main`                      | 永久   | 必须 PR + 2 人 review |
| develop | `develop`                   | 永久   | 必须 PR + 1 人 review |
| feature | `feature/<issue-id>-<简短描述>` | 开发期间 | 必须 PR + 1 人 review |
| hotfix  | `hotfix/x.x.x`              | 修复期间 | 必须 PR + 2 人 review |

### 命名示例

```
# 功能分支
feature/123-sampling-algorithm
feature/124-gait-control
feature/125-sensor-fusion

# 热修复分支
hotfix/1.0.1
hotfix/1.0.2
```

---

## 3. 版本标签规范

### 标签格式

```
v<主版本>.<次版本>.<修正版本>-<标识>
```

### 标签类型说明

| 标签 | 说明 | 使用场景 |
|-----|------|---------|
| `vX.Y.Z-alpha` | 开发测试版本 | 日常开发、集成测试 |
| `vX.Y.Z-rc1` | 候选发布版本 | 正式发布前的预发布版本 |
| `vX.Y.Z-competition` | 比赛使用版本 | 比赛现场部署 |
| `vX.Y.Z-hotfix` | 紧急修复版本 | 赛后或现场紧急修复 |

### 标签示例

```
v1.0.0-alpha           # 开发测试版本
v1.0.0-rc1             # 候选发布版本 1
v1.0.0-rc2             # 候选发布版本 2
v1.0.0-competition     # 比赛使用版本
v1.1.0-hotfix          # 紧急修复版本
```

---

## 4. 分支管理详细流程

### 4.1 开始新功能开发

1. 从 GitHub Projects 看板的 "To Do" 列领取 Issue
2. 更新并创建功能分支

```bash
git checkout develop
git pull origin develop
git checkout -b feature/123-sampling-algorithm
```

3. 开发并频繁 commit

```bash
git add .
git commit -m "feat: 实现采样算法基础框架"
```

4. 推送分支到远程

```bash
git push -u origin feature/123-sampling-algorithm
```

5. 创建 Pull Request（标题格式：`[Feature] #123 <描述>`，关联 Issue：使用 `Closes #123`）
6. 在 GitHub Projects 中将 Issue 从 "To Do" 移至 "In Progress"

### 4.2 代码审查流程

**审查者操作：**

1. 收到 PR 通知后，检出分支查看

```bash
git fetch origin
git checkout feature/123-sampling-algorithm
git log --oneline develop..HEAD
```

2. 运行 CI 检查，确保所有测试通过
3. 进行代码审查：逻辑正确性、编码规范、单元测试覆盖、文档更新
4. 批准或留下评论请求修改

**开发者响应审查反馈：**

1. 根据反馈修改代码

```bash
git add .
git commit -m "fix: 调整采样算法边界条件"
git push
```

2. PR 自动更新，无需新建

### 4.3 合并（Merge）流程

**合并条件（必须全部满足）：**

- CI 所有检查通过
- 至少 1 位 Reviewer 批准（feature）/ 2 位（hotfix）
- 无未解决的评论
- 分支是最新的（无冲突）

**合并操作：**

1. 在 GitHub PR 界面点击 "Squash and merge"
2. 删除源分支（勾选 "Delete branch"）
3. PR 自动关闭，关联的 Issue 移动到 "Done"

### 4.4 发布（Tag）流程

比赛项目使用 Tag 标记版本，无需单独创建 release 分支。

1. 准备发布（确保所有功能分支已合并到 develop）
2. 集成测试：在测试环境进行完整测试
3. 测试通过后合并到 main

```bash
git checkout main
git pull origin main
git merge origin/develop --no-ff -m "Merge develop into main for v1.0.0"
```

4. 打标签并推送

```bash
git tag -a v1.0.0-competition -m "比赛版本 v1.0.0"
git push origin v1.0.0-competition
```

5. 在 GitHub Releases 页面创建 Release，附加变更说明

### 4.5 热修复（Hotfix）流程

当生产环境出现严重 Bug 时：

1. 从 main 创建热修复分支

```bash
git checkout main
git pull origin main
git checkout -b hotfix/1.0.1
```

2. 修复 Bug 并提交

```bash
git add .
git commit -m "fix: 修复传感器数据溢出问题"
```

3. 创建 PR → main 和 develop（两个 PR）
4. 指定 2 位 Reviewer 进行紧急审查
5. 合并后立即部署

---

## 5. 日常开发规范

### 开发前（每日开始工作时）

1. 同步最新代码

```bash
git checkout develop
git pull origin develop
```

2. 查看任务：在 GitHub Projects 看板认领当天任务
3. 创建或切换到工作分支

```bash
# 继续已有分支
git checkout feature/123-xxx
git pull origin feature/123-xxx

# 或创建新分支
git checkout -b feature/124-yyy
```

### 开发中

1. 编写代码，保持小步提交

```bash
git add .
git commit -m "feat(sensor): 实现陀螺仪数据采集"
```

2. 推送代码（建议至少每天一次）

```bash
git push origin feature/123-xxx
```

3. 确保 CI 通过，如有失败及时修复

### 开发后（每日结束前）

1. 提交当天工作

```bash
git add .
git commit -m "fix: 调整步态参数"
git push
```

2. 更新 GitHub Projects：在看板中移动卡片状态，确保 Issue 与 PR 关联正确
3. 如有未完成的工作，在 Issue 下留言说明进度

---

## 6. Commit 规范（Angular 规范）

### 提交信息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 类型说明

| 类型 | 说明 | 示例 |
|-----|------|-----|
| feat | 新功能 | `feat(gait): 添加对角步态算法` |
| fix | Bug 修复 | `fix(sensor): 修复陀螺仪漂移问题` |
| docs | 文档更新 | `docs: 更新接口文档` |
| style | 代码格式（不影响功能的修改） | `style: 格式化代码` |
| refactor | 重构（不修复问题不添加功能） | `refactor(nav): 简化路径规划逻辑` |
| test | 测试相关 | `test: 添加步态单元测试` |
| chore | 构建/工具/依赖更新 | `chore: 更新 CI 配置` |

### 提交示例

```
feat(gait): 添加对角步态算法

实现四足机器人的对角步态行走模式，包括：
- 步态状态机设计
- 腿部相位计算
- 重心转移逻辑

Closes #123
```

---

## 7. 冲突处理方法

### 预防冲突

定期同步 develop，避免冲突累积：

```bash
git checkout feature/xxx
git fetch origin
git merge origin/develop
# 或使用 rebase
git rebase origin/develop
```

### 解决冲突

1. 冲突发生时，手动编辑冲突文件，保留需要的代码，删除 `<<<<<<<`、`=======`、`>>>>>>>` 标记
2. 标记冲突已解决并提交

```bash
git add <resolved-files>
git commit -m "merge: 解决与 develop 的冲突"
# 或继续 rebase
git rebase --continue
```

3. 如使用 rebase，强制推送

```bash
git push --force-with-lease
```

### 大型冲突处理

与相关开发者沟通后处理，约定时间一起解决，避免在 deadline 前进行大规模合并。

---

## 8. GitHub Projects 看板使用说明

### 项目配置

- **项目名称**: Lunar-Grotto-Walker
- **视图**: Sprint 看板（2 周一个 Sprint）
- **列**: To Do | In Progress | In Review | Done

### 看板流程

```
To Do ──────> In Progress ──────> In Review ──────> Done
  │              │                   │                │
  │领取任务       正在开发             等待审查           已完成
  │              │                   │                │
  └──────────────┴───────────────────┴────────────────┘
```

### Issue 创建与关联

1. **创建 Issue**：在 GitHub Issues 页面创建，选择对应模板
2. **关联 Project**：在 Issue 右侧边栏选择 Project
3. **关联 PR**：在 PR 描述中使用 `Closes #123` 或 `Fixes #123`
4. **移动卡片**：在 Project 看板中拖动卡片到对应列

### Issue 模板

1. **功能开发** (feature): 包含功能描述、验收标准、技术方案
2. **Bug 修复** (bug): 包含问题描述、复现步骤、预期行为
3. **文档更新** (documentation): 包含文档位置、更新内容

---

## 附录：简化 Git Flow 分支策略（比赛项目版）

### 核心原则

1. **develop 是主开发分支**：所有功能合并到 develop
2. **main 仅用于稳定版本**：通过 tag 标记发布点
3. **hotfix 直接处理**：紧急修复从 main 创建 hotfix 分支
4. **不设 release 分支**：减少流程开销

### 典型开发周期

```
开发期（赛前 N 周）：
  feature/xxx → develop → (测试) → main + tag → 比赛

比赛期间：
  hotfix/xxx → main + tag → 现场部署
```

---

## 分支策略速查表

| 操作 | 命令 |
|-----|------|
| 更新 develop | `git checkout develop && git pull origin develop` |
| 创建功能分支 | `git checkout -b feature/123-xxx` |
| 推送功能分支 | `git push -u origin feature/123-xxx` |
| 查看分支差异 | `git log --oneline develop..HEAD` |
| 合并 develop | `git merge origin/develop` |
| 创建 Tag | `git tag -a v1.0.0-competition -m "message"` |
| 推送 Tag | `git push origin v1.0.0-competition` |
| 创建热修复分支 | `git checkout -b hotfix/1.0.1 origin/main` |

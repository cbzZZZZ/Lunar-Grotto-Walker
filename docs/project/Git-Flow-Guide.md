# Git Flow 开发流程指南

## 1. 项目概述

本项目为月球熔岩洞机器人智能自主采样任务，采用 Git Flow 模型结合 GitHub Projects (Beta) 进行项目管理，适用于 4-10 人团队的 C/C++（嵌入式）+ Python（AI）混合技术栈项目。

### 技术栈

- **代码托管**: GitHub
- **项目管理**: GitHub Projects (Beta) - Sprint 看板
- **CI/CD**: GitHub Actions
- **嵌入式**: C/C++（编译检查、静态分析、单元测试）
- **AI/仿真**: Python（代码格式、lint、单元测试）

### 分支结构概览

```
feature/123-xxx  ─┐
feature/124-yyy  ─┤
                  ├──> develop ──────────────> main
                  │                              ↑
hotfix/x.x.x     ─┴──────────────────────────────┘
```

---

## 2. 分支命名规范

| 分支类型 | 命名规范 | 生命周期 | 保护规则 |
|---------|---------|---------|---------|
| main | `main` | 永久 | 必须 PR + 2 人 review |
| develop | `develop` | 永久 | 必须 PR + 1 人 review |
| feature | `feature/<issue-id>-<简短描述>` | 开发期间 | 必须 PR + 1 人 review |
| hotfix | `hotfix/x.x.x` | 修复期间 | 必须 PR + 2 人 review |

### 命名示例

```
# 功能分支
feature/123-sampling-algorithm
feature/124-gait-control
feature/125-sensor-fusion

# 热修复分支
hotfix/1.0.1-critical-fix
hotfix/1.0.2-sensor-overflow
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

```
1. 从 GitHub Projects 看板的 "To Do" 列领取 Issue（或由 SM 分配）
2. 在本地更新 develop 分支：
   git checkout develop
   git pull origin develop
3. 创建功能分支（命名规范：feature/<issue编号>-<简短描述>）：
   git checkout -b feature/123-sampling-algorithm
4. 在分支上进行开发，频繁 commit：
   git add .
   git commit -m "feat: 实现采样算法基础框架"
5. 推送分支到远程：
   git push -u origin feature/123-sampling-algorithm
6. 创建 Pull Request：
   - 标题格式：[Feature] #<issue编号> <简短描述>
   - 内容使用 PR 模板
   - 关联对应 Issue（使用 keywords: closes #123）
   - 指定 Reviewer
7. 在 GitHub Projects 中将 Issue 从 "To Do" 移至 "In Progress"
```

### 4.2 代码审查流程

**审查者（Reviewer）操作：**

```
1. 收到 PR 通知后，在本地检出分支查看：
   git fetch origin
   git checkout feature/123-sampling-algorithm
   git log --oneline develop..HEAD  # 查看提交历史
2. 运行 CI 检查，确保所有测试通过
3. 进行代码审查，检查：
   - 代码逻辑是否正确
   - 是否符合编码规范
   - 是否有单元测试覆盖
   - 文档是否更新
4. 留下评论，必要时要求修改
5. 批准或请求修改
```

**开发者响应审查反馈：**

```
1. 根据反馈修改代码：
   git add .
   git commit -m "fix: 调整采样算法边界条件"
   git push
2. PR 自动更新，无需新建
```

### 4.3 合并（Merge）流程

**合并条件（必须全部满足）：**

- CI 所有检查通过
- 至少 1 位 Reviewer 批准（feature）/ 2 位（hotfix）
- 无未解决的评论
- 分支是最新的（无冲突）

**合并操作：**

```
1. 在 GitHub PR 界面点击 "Squash and merge"
2. 删除源分支（勾选 "Delete branch"）
3. PR 自动关闭，关联的 Issue 自动转为 Done
4. 在 GitHub Projects 中手动将卡片移动到 "Done" 列
```

### 4.4 发布（Tag）流程

比赛项目使用 Tag 标记版本，无需单独创建 release 分支。

```
1. 准备发布（由 Release Manager 执行）：
   git checkout develop
   git pull
   # 确保所有功能分支已合并

2. 集成测试：在测试环境进行完整测试

3. 测试通过后合并到 main：
   git checkout main
   git merge develop --no-ff -m "Merge develop into main for v1.0.0"

4. 打标签：
   git tag -a v1.0.0-competition -m "比赛版本 v1.0.0"
   git push origin v1.0.0-competition

5. 在 GitHub Releases 页面创建 Release，附加变更说明
```

### 4.5 热修复（Hotfix）流程

当生产环境出现严重 Bug 时：

```
1. 从 main 创建热修复分支：
   git checkout main
   git pull
   git checkout -b hotfix/1.0.1-critical-fix

2. 修复 Bug 并提交（使用 "fix:" 前缀）：
   git commit -m "fix: 修复传感器数据溢出问题"

3. 创建 PR → main 和 develop（两个 PR）

4. 指定 2 位 Reviewer 进行紧急审查

5. 合并后立即部署
```

---

## 5. 日常开发规范

### 开发前（每日开始工作时）

```
1. 同步最新代码：
   git checkout develop
   git pull origin develop

2. 查看任务：在 GitHub Projects 看板认领当天任务

3. 创建/切换到工作分支：
   # 继续开发已有分支
   git checkout feature/123-xxx
   git pull origin feature/123-xxx  # 同步他人可能的更新

   # 或创建新分支
   git checkout -b feature/124-yyy
```

### 开发中

```
1. 编写代码，保持小步提交：
   git add .
   git commit -m "feat(sensor): 实现陀螺仪数据采集"

2. 推送代码（建议至少每天一次）：
   git push origin feature/123-xxx

3. 确保 CI 通过，如有失败及时修复
```

### 开发后（每日结束前）

```
1. 提交当天工作：
   git add .
   git commit -m "fix: 调整步态参数"

2. 推送到远程：
   git push

3. 更新 GitHub Projects：
   - 在看板中移动卡片状态
   - 确保 Issue 与 PR 关联正确

4. 如有未完成的工作，在 Issue 下留言说明进度
```

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

### Commit 规范详细说明

| 类型 | 何时使用 | 是否破坏性 |
|-----|---------|-----------|
| feat | 添加新功能时 | 否 |
| fix | 修复 bug 时 | 否 |
| docs | 仅文档更改时 | 否 |
| style | 代码格式、逗号、空格等 | 否 |
| refactor | 重构代码时 | 否 |
| test | 添加或修改测试时 | 否 |
| chore | 构建流程、辅助工具、依赖变更 | 否 |

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

```
1. 定期同步 develop，避免冲突累积：
   git checkout feature/xxx
   git fetch origin
   git merge origin/develop
   # 或使用 rebase：git rebase origin/develop
```

### 解决冲突

```
1. 冲突发生时，手动编辑冲突文件：
   - 查找 <<<<<<< HEAD
   - 保留需要的代码
   - 删除 <<<<<<<、=======、>>>>>>> 标记

2. 标记冲突已解决：
   git add <resolved-files>

3. 提交冲突解决：
   git commit -m "merge: 解决与 develop 的冲突"
   # 或继续 rebase：git rebase --continue

4. 强制推送（如使用 rebase）：
   git push --force-with-lease
```

### 大型冲突处理

```
1. 与相关开发者沟通后处理
2. 约定时间一起解决冲突
3. 避免在 deadline 前进行大规模合并
```

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

### 赛前流程示例

```
第 1 周：git checkout develop
第 2 周：git merge feature/gait-algorithm
第 3 周：git merge feature/sensor-fusion
第 4 周（赛前）：git checkout main && git merge develop --no-ff
               git tag -a v1.0.0-competition
               git push --tags
比赛当天：直接使用 main 分支代码或现场 hotfix
```

---

## 分支策略速查表

| 操作 | 命令 |
|-----|------|
| 更新 develop | `git checkout develop && git pull` |
| 创建功能分支 | `git checkout -b feature/123-xxx` |
| 推送功能分支 | `git push -u origin feature/123-xxx` |
| 查看分支差异 | `git log --oneline develop..HEAD` |
| 合并 develop | `git merge origin/develop` |
| 创建 Tag | `git tag -a v1.0.0-competition -m "message"` |
| 推送 Tag | `git push origin v1.0.0-competition` |
| 创建热修复分支 | `git checkout -b hotfix/1.0.1 origin/main` |

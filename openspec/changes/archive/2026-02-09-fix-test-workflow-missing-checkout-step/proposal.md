# Change: 修复测试工作流缺少checkout步骤

**Status: ExecutionCompleted** ✅

## Why

测试工作流 `test-notify.yml` 执行失败，错误信息显示无法找到 `action.yml` 文件。根本原因是工作流缺少 `actions/checkout@v4` 步骤来检出仓库代码，导致 GitHub Actions runner 无法访问本地 action 文件。

## What Changes

- 在 `.github/workflows/test-notify.yml` 工作流中添加 `actions/checkout@v4` 步骤
- checkout 步骤将作为第一个执行步骤，确保后续步骤能够访问仓库文件
- 修改位置：第 32-34 行之间（在 `uses: ./` 步骤之前）

## Impact

**受影响的代码：**
- `.github/workflows/test-notify.yml` - 添加 checkout 步骤

**对外部用户的影响：**
- **无影响** - 外部用户通过 `uses: HagiCode-org/haginotifier@v1` 引用 action，不受此变更影响
- 测试工作流仅用于开发者手动测试 action 功能
- action 本身（`action.yml`）未被修改，功能保持不变

**预期效果：**
- 测试工作流能够成功找到 `action.yml` 文件
- Node.js 设置、依赖安装和通知执行步骤将正常运行
- 飞书通知能够正常发送
- GitHub Step Summary 将显示完整的通知结果

**验证方法：**
1. 推送修改后的工作流到仓库
2. 在 GitHub Actions 页面手动触发 "Test Notification" 工作流
3. 确认工作流成功完成
4. 检查飞书群聊是否收到测试消息
5. 查看 GitHub Step Summary 中的输出参数

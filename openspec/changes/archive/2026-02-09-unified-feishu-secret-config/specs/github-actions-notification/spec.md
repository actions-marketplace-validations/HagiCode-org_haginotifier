## ADDED Requirements

### Requirement: Organization-Level Secret Configuration
对于使用 GitHub 组织的仓库，系统 SHALL 支持组织级 Secret 配置，实现一处配置、多处共享的目标。

#### Scenario: 组织管理员配置组织级 Secret
- **GIVEN** GitHub 组织管理员权限
- **WHEN** 在组织设置中创建名为 `FEISHU_WEBHOOK_URL` 的组织 Secret
- **THEN** 该 Secret 可被组织内所有授权仓库直接引用
- **AND** 各子仓库无需单独配置该 Secret

#### Scenario: 子仓库调用可复用 workflow 使用组织级 Secret
- **GIVEN** 组织已配置 `FEISHU_WEBHOOK_URL` 组织级 Secret
- **WHEN** 子仓库调用可复用 workflow
- **AND** 在 `secrets:` 中传递组织级 Secret
- **THEN** workflow 成功执行并发送飞书通知
- **AND** 子仓库无需单独配置 Secret

#### Scenario: 更新组织级 Secret 应用到所有授权仓库
- **GIVEN** 多个仓库正在使用组织级 `FEISHU_WEBHOOK_URL` Secret
- **WHEN** 组织管理员更新该 Secret 的值
- **THEN** 所有授权仓库的下一次 workflow 运行将使用新的 Secret 值
- **AND** 无需逐个更新各仓库的 Secret 配置

## MODIFIED Requirements

### Requirement: GitHub Actions Notification Integration
系统 SHALL 提供可被其他 GitHub Actions workflow 调用的通知能力，支持组织级 Secret 配置方式。

#### Scenario: 使用传统方式调用可复用 workflow（每个仓库单独配置 Secret）
- **GIVEN** 调用方仓库已配置 `FEISHU_WEBHOOK_URL` Secret
- **WHEN** 使用 `uses:` 语法调用可复用 workflow
- **AND** 通过 `secrets:` 传递 Secret
- **THEN** workflow 成功执行并发送飞书通知

#### Scenario: 使用组织级 Secret 调用可复用 workflow
- **GIVEN** 组织已配置 `FEISHU_WEBHOOK_URL` 组织级 Secret
- **WHEN** 子仓库调用可复用 workflow
- **AND** 在 `secrets:` 中引用组织级 Secret
- **THEN** workflow 成功执行并发送飞书通知
- **AND** 子仓库无需单独配置 Secret

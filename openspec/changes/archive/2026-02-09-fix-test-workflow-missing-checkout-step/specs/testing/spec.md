## ADDED Requirements

### Requirement: Test Workflow Repository Checkout
The test workflow SHALL include a checkout step to clone the repository content before executing the local action.

#### Scenario: Successful test workflow execution
- **WHEN** the "Test Notification" workflow is triggered
- **THEN** the workflow SHALL first execute `actions/checkout@v4` to clone the repository
- **AND** the workflow SHALL be able to locate `action.yml` in the runner's working directory
- **AND** the subsequent notification step SHALL execute successfully

#### Scenario: Workflow step ordering
- **WHEN** the test workflow executes
- **THEN** the checkout step SHALL be the first step in the steps list
- **AND** the checkout step SHALL complete before the notification step (`uses: ./`)

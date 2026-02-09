# haginotifier

> Reusable GitHub Actions workflow for sending notifications to Feishu (飞书)

A simple, reusable GitHub Actions workflow that allows any repository to send notifications to Feishu via webhook. This provides a unified notification mechanism across your organization, eliminating the need to duplicate notification logic in each repository.

## Features

- **Reusable Workflow**: Call from any repository using standard `uses:` syntax
- **Multiple Message Types**: Support for text, rich text (post), and interactive card messages
- **Simple Configuration**: Just provide a webhook URL and message content
- **Standardized Output**: Returns status, timestamp, and response data for downstream processing
- **ESM Module**: Built with modern TypeScript and ESM for Node.js 18+

## Usage

### Basic Example

Send a simple text message:

```yaml
jobs:
  notify:
    uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@main
    with:
      message: 'Deployment successful!'
    secrets:
      FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
```

### Rich Text Message

Send a rich text message with a title:

```yaml
jobs:
  notify:
    uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@main
    with:
      message: 'The production deployment has been completed successfully.'
      msg_type: 'post'
      title: 'Deployment Notification'
    secrets:
      FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
```

### Interactive Card Message

Send an interactive card message:

```yaml
jobs:
  notify:
    uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@main
    with:
      message: 'Build #123 completed in 5 minutes'
      msg_type: 'interactive'
      title: 'CI/CD Pipeline Status'
    secrets:
      FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
```

### Using Outputs

Access the notification result in subsequent steps:

```yaml
jobs:
  notify:
    uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@main
    with:
      message: 'Test notification'
    secrets:
      FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
  check-result:
    needs: notify
    runs-on: ubuntu-latest
    steps:
      - name: Check notification status
        run: |
          echo "Status: ${{ needs.notify.outputs.status }}"
          echo "Timestamp: ${{ needs.notify.outputs.timestamp }}"
          echo "Response: ${{ needs.notify.outputs.response }}"
```

## Input Parameters

| Parameter | Required | Type | Description | Default |
|-----------|----------|------|-------------|---------|
| `message` | Yes | string | Notification message content | - |
| `msg_type` | No | string | Message type: `text`, `post`, or `interactive` | `text` |
| `title` | No | string | Message title for post/interactive messages | - |

## Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `FEISHU_WEBHOOK_URL` | Yes | Feishu Webhook URL for sending notifications |

## Output Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Notification delivery status: `success` or `failure` |
| `timestamp` | string | ISO 8601 timestamp of notification send attempt |
| `response` | string | Webhook response content or error message |

## Setup

### Option 1: Organization-level Secret (Recommended for GitHub Organizations)

Configure the webhook URL once at the organization level, and all repositories within the organization can reuse it without managing their own webhook URL. This is the **recommended approach** for managing multiple repositories.

**Prerequisites:**
- You must be a member of a GitHub organization
- You need organization admin permissions to configure organization secrets

**To configure organization-level Secret:**

1. Navigate to your organization's main page on GitHub
2. Click on **Settings** tab
3. In the left sidebar, select **Secrets and variables** > **Actions**
4. Click **New repository secret**
5. Name the secret `FEISHU_WEBHOOK_URL`
6. Paste your Feishu Webhook URL as the value
7. **Important**: Configure **Access settings** to select which repositories can use this secret
8. Click **Add secret**

**In consuming repositories:**

```yaml
jobs:
  notify:
    uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@main
    with:
      message: 'Your notification here'
    secrets:
      FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
```

**Benefits of Organization-level Secrets:**
- Configure once, use across multiple repositories
- Easier management - update the secret in one place
- Official GitHub recommended approach
- More secure with centralized control

### Option 2: Per-Repository Configuration

Each repository manages its own webhook URL. This is suitable for:
- Personal accounts without organization access
- Repositories that need different webhook URLs

1. Go to **Settings** > **Secrets and variables** > **Actions** in your repository
2. Create a new secret named `FEISHU_WEBHOOK_URL`
3. Paste your Feishu Webhook URL as the value

### Creating a Feishu Webhook

1. Open your Feishu group chat
2. Go to Group Settings > Group Chat > Chatbots > Add Robot
3. Select "Custom Bot" and configure it
4. Copy the Webhook URL

## Migration Guide

### Migrating from Per-Repository to Organization-level Secret

If you currently have the `FEISHU_WEBHOOK_URL` secret configured in multiple repositories, you can migrate to organization-level secrets for easier management.

**Step-by-step migration:**

1. **Create the organization-level secret** (see Option 1 above)
2. **Configure access settings** to select which repositories can use the secret
3. **Verify** that at least one repository's workflow runs successfully
4. **Optional cleanup**: Remove the `FEISHU_WEBHOOK_URL` secret from individual repositories

**Migration verification checklist:**

- [ ] Organization-level secret `FEISHU_WEBHOOK_URL` has been created
- [ ] Access settings are configured for the correct repositories
- [ ] At least one workflow using the organization secret runs successfully
- [ ] Notification is received in Feishu

**Rollback plan:**

If you encounter any issues, you can easily rollback:
1. Re-create the `FEISHU_WEBHOOK_URL` secret in individual repositories
2. No changes to workflow files are needed

### 3. Reference a Specific Version

For production use, pin to a specific version tag instead of `main`:

```yaml
uses: HagiCode-org/haginotifier/.github/workflows/notify.yml@v1.0.0
```

## Development

### Prerequisites

- Node.js 18 or higher
- tsx (TypeScript executor)

### Local Testing

```bash
# Install dependencies
npm install

# Run locally with tsx
FEISHU_WEBHOOK_URL="your_webhook_url" FEISHU_MESSAGE="Test message" npx tsx src/feishu.ts
```

### Project Structure

```
haginotifier/
├── .github/
│   └── workflows/
│       ├── notify.yml         # Reusable workflow definition
│       └── test-notify.yml    # Manual test workflow
├── src/
│   ├── feishu.ts              # Feishu Webhook implementation
│   ├── types.ts               # TypeScript type definitions
│   └── index.ts               # Main entry point
├── package.json
├── tsconfig.json
└── README.md
```

### Testing

You can test the notification workflow directly from GitHub:

1. Go to Actions tab in the haginotifier repository
2. Select "Test Notification" workflow
3. Click "Run workflow"
4. Fill in the parameters:
   - **message**: Your test message content
   - **msg_type**: Choose "text", "post", or "interactive"
   - **title**: (Optional) Title for post/interactive messages
5. Click "Run workflow"

The test will send a notification and display the result in the workflow summary.

## Extensibility

The codebase is designed to be easily extended with additional notification providers. To add a new provider (e.g., DingTalk, WeCom):

1. Create a new module (e.g., `src/dingtalk.ts`)
2. Implement a similar `sendNotification` function
3. Add provider-specific types to `src/types.ts`
4. Export from `src/index.ts`
5. Update the workflow to accept a `provider` input parameter

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## FAQ

**Q: Can I use organization-level secrets with personal repositories?**

A: No, organization-level secrets are only available for GitHub organizations. If you're using a personal account, use the per-repository configuration option (Option 2).

**Q: Do I need to modify my workflow files when migrating to organization-level secrets?**

A: No changes are needed. The workflow configuration remains exactly the same. You just need to configure the secret at the organization level instead of in individual repositories.

**Q: What happens if a repository doesn't have access to the organization-level secret?**

A: The workflow will fail with an error indicating that the secret is not available. Make sure to configure the access settings when creating the organization-level secret.

**Q: Can I mix organization-level and per-repository secrets?**

A: Yes, you can use both approaches simultaneously. Some repositories can use the organization-level secret while others manage their own. This is useful during migration.

**Q: How do I update the webhook URL for all repositories?**

A: With organization-level secrets, simply update the secret value in the organization settings. All repositories with access will automatically use the new value.

## Troubleshooting

### Secret not found error

**Symptom**: Workflow fails with error about `FEISHU_WEBHOOK_URL` not being found.

**Solutions**:
- Verify the secret exists at the organization or repository level
- For organization secrets: Check access settings to ensure the repository has access
- Check that the secret name is spelled correctly (case-sensitive)

### Webhook authentication failed

**Symptom**: Workflow runs but notification is not received, or webhook returns authentication error.

**Solutions**:
- Verify the webhook URL is correct and complete
- Check that the custom bot in Feishu is still enabled
- Ensure the webhook hasn't been regenerated or changed in Feishu

### Access denied for organization secret

**Symptom**: Error indicating the repository cannot access the organization-level secret.

**Solutions**:
- Contact your organization administrator to grant access
- Check the secret's access settings in organization settings
- Verify the repository is part of the same organization

### Notification format issues

**Symptom**: Notification is received but formatting is incorrect.

**Solutions**:
- Verify `msg_type` parameter is set correctly (`text`, `post`, or `interactive`)
- Ensure `title` is provided when using `post` or `interactive` message types
- Check that message content follows Feishu's message format requirements

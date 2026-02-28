# action-aws-nuke

A GitHub Action to run [aws-nuke](https://github.com/ekristen/aws-nuke) for cleaning up AWS account resources.

## Warning

This action will potentially delete AWS resources. If you do not have experience
with aws-nuke be extra careful. I can not be responsible for your mistakes.

## Usage

```yaml
- uses: webframp/action-aws-nuke@v1
  with:
    config-file: 'config/aws-nuke-config.yml'
    dry-run: 'true'
```

## Prerequisites

This action requires AWS credentials to be configured before use. The recommended approach is [OIDC authentication](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services):

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/my-nuke-role
    aws-region: us-east-1

- uses: webframp/action-aws-nuke@v1
  with:
    config-file: 'config/aws-nuke-config.yml'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `config-file` | Path to aws-nuke config file | Yes | - |
| `dry-run` | Run in dry-run mode | No | `true` |
| `version` | aws-nuke version | No | `3.64.0` |

## Outputs

| Output | Description |
|--------|-------------|
| `resources-found` | Whether resources were found (`true`/`false`) |
| `resources-count` | Number of resources found or deleted |
| `output-file` | Path to aws-nuke output log |

## Examples

### Dry-run with notification

```yaml
name: AWS Nuke Dry Run

on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  dry-run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/nuke-role
          aws-region: us-east-1

      - uses: webframp/action-aws-nuke@v1
        id: nuke
        with:
          config-file: 'config/aws-nuke-config.yml'
          dry-run: 'true'

      - name: Report results
        if: steps.nuke.outputs.resources-found == 'true'
        run: |
          echo "Found ${{ steps.nuke.outputs.resources-count }} resources to clean up"
          cat "${{ steps.nuke.outputs.output-file }}"
```

### Actual deletion (use with caution)

```yaml
- uses: webframp/action-aws-nuke@v1
  with:
    config-file: 'config/aws-nuke-config.yml'
    dry-run: 'false'
```

## aws-nuke Configuration

This action requires a valid aws-nuke configuration file. See the [aws-nuke documentation](https://ekristen.github.io/aws-nuke/) for configuration options.

Minimal example:

```yaml
regions:
  - us-east-1
  - global

account-blocklist:
  - "999999999999"  # Production - never touch

accounts:
  "123456789012":
    filters:
      S3Bucket:
        - "my-protected-bucket"
      IAMRole:
        - type: contains
          value: "github"
```

## Security Considerations

- **Always use dry-run first** to verify what will be deleted
- **Use account blocklist** to protect production accounts
- **Use resource filters** to protect critical infrastructure
- **Use IAM deny statements** as defense-in-depth against misconfigurations
- **Require manual approval** before actual deletion in CI/CD workflows

## License

Apache 2.0

# Get AWS Credentials

A reusable GitHub Action that assumes an **IAM role chain** using OpenID Connect (OIDC) to access AWS resources across accounts.

This action:
1. Uses OIDC to authenticate into a **service account** role in the central AWS account.
2. Assumes a **target role** in the specified destination AWS account.
3. Exports short-lived AWS credentials automatically to all subsequent workflow steps.

---

## How It Works

```
GitHub → OIDC → Service Account Role (in central AWS account)
         → Assume Role → Target Account Role
```

The service account role name follows the pattern:
```
GHAFederatedRole-<account-id>
```
and the target role in the destination account is specified by `target-role-name`.

---

## Inputs

| Name | Description | Required | Default |
|------|--------------|-----------|----------|
| `account-id` | AWS Account ID of the target environment | Yes | — |
| `region` | AWS Region | No | `us-east-1` |
| `target-role-name` | Name of the target role to assume in the target account | No | `DefaultCrossAccountRole` |

---

## Example Usage

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Required for OIDC
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Get AWS Credentials
        uses: TRI-Actions/get-aws-credentials@v1
        with:
          account-id: 222222222222

      - name: Verify AWS identity
        run: aws sts get-caller-identity

      - name: Deploy resources
        run: |
          aws s3 ls
          aws cloudformation describe-stacks
```

After the “Get AWS Credentials” step runs, the environment automatically contains:
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```
You can use AWS CLI or SDK commands directly without additional configuration.

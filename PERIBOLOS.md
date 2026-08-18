# Peribolos Organization Management

This directory contains the declarative configuration for managing the `open-component-model` GitHub organization using [Peribolos](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/).

## Files

- **`peribolos.yaml`**: Declarative configuration defining organization settings, members, admins, and teams
- **`.github/workflows/peribolos.yml`**: GitHub Actions workflow that syncs the organization state

## How It Works

Peribolos reads the `peribolos.yaml` configuration and synchronizes the GitHub organization to match the desired state. This includes:

- Organization settings (name, description, default permissions, etc.)
- Organization members and their roles (admin vs. member)
- Teams, team members, and team maintainers
- Team privacy settings

## Workflow Behavior

The GitHub Actions workflow runs in the following scenarios:

1. **Daily at 3:17 AM UTC** - Scheduled automatic sync (dry-run only)
2. **On push to main** - When `peribolos.yaml` or the workflow file changes (dry-run only)
3. **Manual trigger** - Via workflow_dispatch with option to apply changes

### Dry-Run vs. Apply

By default, all automated runs use **dry-run mode** (`--confirm=false`), which:
- ✅ Shows what changes would be made
- ❌ Does NOT actually apply any changes
- Safe for scheduled and automatic runs

To **apply changes**, you must:
1. Go to Actions → Peribolos Organization Sync
2. Click "Run workflow"
3. Select "Apply changes (true)" from the dropdown
4. Click "Run workflow"

## Safety Features

The workflow includes several safety mechanisms:

- `--require-self=true`: Ensures the token owner remains in the organization
- `--min-admins=2`: Requires at least 2 admins in the organization
- `--maximum-removal-delta=0.3`: Prevents removing more than 30% of members in one run

## Required Secret

The workflow requires a GitHub token with organization admin permissions:

**Secret name:** `PERIBOLOS_GITHUB_TOKEN`

**Required permissions:**
- `admin:org` (full organization control)

To create the token:
1. Go to Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Create a token with organization administration permissions
3. Add it as a repository secret named `PERIBOLOS_GITHUB_TOKEN`

## Making Changes

1. Edit `peribolos.yaml` to add/remove members, admins, or teams
2. Commit and push to main branch
3. The workflow will run in dry-run mode and show proposed changes
4. Review the workflow logs
5. Manually trigger the workflow with `confirm=true` to apply changes

## Dumping Current State

To export the current organization state:

```bash
# Install peribolos
go install sigs.k8s.io/prow/cmd/peribolos@latest

# Dump current state
peribolos --dump open-component-model \
  --github-token-path <(echo "$GITHUB_TOKEN") \
  > peribolos-current.yaml
```

## Reference

- [Peribolos Documentation](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/)
- [Prow GitHub](https://github.com/kubernetes/test-infra/tree/master/prow)

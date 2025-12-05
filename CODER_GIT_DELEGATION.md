# Coder Git Delegation Support

This document describes the changes made to support Coder's git delegation mechanism in doodba-copier-template.

## Problem

When using doodba within a Coder workspace, git operations inside Docker containers couldn't access private repositories even though the host had access via Coder's git delegation. Previously, users had to manually copy SSH keys to `odoo/custom/ssh/`, which is inconvenient and needs to be repeated for each new workspace.

## Solution

Pass through Coder's git delegation environment variables to Docker containers, allowing authenticated git operations without copying SSH keys.

## Changes Made

### 1. `setup-devel.yaml.jinja`

**Environment Variables:**
```yaml
# Coder git delegation for authenticated repositories
GIT_SSH_COMMAND: "${GIT_SSH_COMMAND:-}"
GIT_ASKPASS: "${GIT_ASKPASS:-}"
GIT_AUTHOR_NAME: "${GIT_AUTHOR_NAME:-}"
GIT_AUTHOR_EMAIL: "${GIT_AUTHOR_EMAIL:-}"
GIT_COMMITTER_NAME: "${GIT_COMMITTER_NAME:-}"
GIT_COMMITTER_EMAIL: "${GIT_COMMITTER_EMAIL:-}"
```

### 2. `devel.yaml.jinja`

**Environment Variables:**
```yaml
# Coder git delegation for authenticated repositories
GIT_SSH_COMMAND: "${GIT_SSH_COMMAND:-}"
GIT_ASKPASS: "${GIT_ASKPASS:-}"
GIT_AUTHOR_NAME: "${GIT_AUTHOR_NAME:-}"
GIT_AUTHOR_EMAIL: "${GIT_AUTHOR_EMAIL:-}"
GIT_COMMITTER_NAME: "${GIT_COMMITTER_NAME:-}"
GIT_COMMITTER_EMAIL: "${GIT_COMMITTER_EMAIL:-}"
```

### 3. `aggregate.sh.jinja`

A convenience script that automatically:
- Sets required ownership variables
- Runs gitaggregator

The git environment variables (GIT_SSH_COMMAND, etc.) are automatically inherited from the host environment.

### 4. `tasks_downstream.py`

Updated the `git_aggregate` task to pass through git environment variables when running docker-compose.

## Usage

### In Coder Workspaces

Simply run:
```bash
./aggregate.sh
```

### Using invoke

Run:
```bash
invoke git-aggregate
```

The task will automatically pass through git environment variables if they exist.

### In Non-Coder Environments

The changes are backward compatible. If git environment variables are not set, the containers fall back to standard SSH authentication (keys in `odoo/custom/ssh/`).

## Benefits

✅ No manual SSH key copying
✅ Automatic authentication in Coder workspaces
✅ Works across all workspace recreations
✅ Backward compatible with non-Coder environments
✅ More secure (credentials managed by Coder)

## Testing

To test these changes:

1. Ensure you're in a Coder workspace with git delegation enabled
2. Verify `GIT_SSH_COMMAND` environment variable is set
3. Run `./aggregate.sh`
4. Verify that private repositories (like `git@github.com:odoo/enterprise.git`) are cloned successfully

## Contributing

To submit these changes to the upstream doodba-copier-template:

1. Create a branch in your fork
2. Commit these changes
3. Open a pull request to Tecnativa/doodba-copier-template
4. Reference this document in the PR description

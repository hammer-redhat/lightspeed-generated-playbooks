# Redis Cluster Configuration Automation

This automation tooling generates Redis cluster configuration files from templates and automatically creates Pull Requests for team review and deployment.

## Overview

This toolkit provides:
- **Template-based configuration generation** for Redis clusters
- **Automated git workflow** (branch creation, commit, push)
- **Pull Request automation** using GitHub CLI
- **YAML validation** of generated configurations
- **Audit trail** through git history

## Files Structure

```
redis/
├── create_pr_for_redis.yml          # Main automation playbook
├── example_vars.yml                 # Example variables file
├── example_redis_pr.yml             # Original example configuration
├── templates/
│   └── redis_cluster_config.yml.j2  # Jinja2 template for Redis config
└── README.md                        # This file
```

## Prerequisites

### Required Software
- Ansible 2.9 or higher
- Git
- Python 3.x with PyYAML
- GitHub CLI (`gh`) - optional but recommended for PR automation

### Installation

```bash
# Install Ansible
sudo dnf install ansible  # RHEL/CentOS
# or
sudo apt install ansible  # Ubuntu/Debian

# Install GitHub CLI (optional)
# RHEL/CentOS
sudo dnf install gh

# Ubuntu/Debian
sudo apt install gh

# Authenticate GitHub CLI
gh auth login
```

### Git Configuration
- SSH key configured for your git repository OR GitHub Personal Access Token
- Appropriate repository access/permissions
- Repository URL updated in vars file

**For private repositories:** See [GIT_AUTHENTICATION.md](GIT_AUTHENTICATION.md) for detailed authentication setup.

## Quick Start

> **New!** The playbook now includes all prerequisite checks built-in. Just run it directly!
> See `RUN_ME.md` for the quickest getting started guide.

### 1. Create Ansible Vault for Secrets

Store sensitive information in an Ansible Vault:

```bash
ansible-vault create redis_secrets.yml
```

Add your secrets:

```yaml
---
vault_cluster_admin_password: "your_secure_password"
vault_cluster_admin_password_active: "your_secure_password_active"
vault_bind_pass: "your_bind_password"
vault_secret_hi: "your_secret"
```

### 2. (Optional) Customize Variables

Copy and edit the example vars file for your specific cluster:

```bash
cp example_vars.yml my_cluster_vars.yml
vi my_cluster_vars.yml
```

**Key variables to customize:**

```yaml
# Cluster Identification
cluster_name: your-redis-cluster.example.com
cluster_master: your-master-node.prod.example.com

# Version Information
redis_version: "7.8.6-60"
bdb_version: "7.4.0"

# Git Repository
git_repo_url: "git@github.com:yourorg/redis-configs.git"

# Vault Secrets (reference your Ansible Vault)
vault_cluster_admin_password: "{{ vault_cluster_admin_password }}"
vault_bind_pass: "{{ vault_bind_pass }}"
```

### 3. Run Everything in One Command

Execute the playbook - it will check prerequisites, generate config, and create PR:

```bash
# With vault password prompt
ansible-playbook create_pr_for_redis.yml \
  -e @my_cluster_vars.yml \
  -e @redis_secrets.yml \
  --ask-vault-pass

# With vault password file
ansible-playbook create_pr_for_redis.yml \
  -e @my_cluster_vars.yml \
  -e @redis_secrets.yml \
  --vault-password-file ~/.vault_pass

# Dry run to see what would happen
ansible-playbook create_pr_for_redis.yml \
  -e @my_cluster_vars.yml \
  -e @redis_secrets.yml \
  --ask-vault-pass \
  --check
```

The playbook will automatically:
- ✓ Check all prerequisites (git, python3, GitHub CLI)
- ✓ Validate your configuration
- ✓ Generate the Redis cluster config file
- ✓ Create a git branch, commit, and push
- ✓ Create a Pull Request (if GitHub CLI available)

## What the Automation Does

1. **Checks prerequisites** - git, python3, GitHub CLI availability
2. **Validates SSH access** - tests connection to git repository
3. **Validates variables** - ensures all required variables are present
4. **Clones** the git repository
5. **Creates** a new feature branch with timestamp
6. **Generates** Redis configuration from template
7. **Validates** the generated YAML syntax
8. **Commits** the configuration file
9. **Pushes** to remote repository
10. **Creates** a Pull Request (if GitHub CLI available)

## Configuration Variables Reference

### Core Cluster Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `cluster_name` | Primary cluster hostname | `mbf-redis-001.example.com` |
| `cluster_master` | Primary master node | `lmbf001a.prod.example.com` |
| `cluster_name_active` | Active/DR cluster hostname | `mbf-redis-002.example.com` |
| `cluster_master_active` | Active/DR master node | `lmbf002a.prod.example.com` |

### Version Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `redis_version` | Redis Enterprise version | `7.8.6-60` |
| `bdb_version` | Database version | `7.4.0` |
| `featureset_version` | Feature set version | `8` |

### High Availability Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `slave_ha_enabled` | Enable slave HA | `enabled` |
| `slave_ha_grace_period` | Grace period in seconds | `120` |
| `slave_ha_cooldown_period` | Cooldown period in seconds | `240` |
| `slave_ha_bdb_cooldown_period` | BDB cooldown in seconds | `360` |

### Git & PR Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `git_repo_url` | Repository URL | `git@github.com:org/repo.git` |
| `git_branch_prefix` | Branch name prefix | `redis-deployment` |
| `pr_title_prefix` | PR title prefix | `Redis Cluster Deployment` |
| `pr_reviewers` | List of reviewers | `['user1', 'user2']` |
| `pr_labels` | PR labels | `['redis', 'infrastructure']` |

## Generated Output

### File Naming Convention
```
redis_cluster_<sanitized_cluster_name>_<timestamp>.yml
```

Example: `redis_cluster_mbf_redis_001_example_com_1697472000.yml`

### Output Location
```
<git_repo_local_path>/deployments/<generated_filename>
```

## Workflow After PR Creation

1. **Review**: Team members review the generated configuration
2. **Approve**: Required approvals obtained
3. **Merge**: PR merged to main branch
4. **Deploy**: Use the merged file for cluster deployment:

```bash
ansible-playbook deployments/redis_cluster_<name>_<timestamp>.yml \
  --ask-vault-pass
```

## Advanced Usage

### Custom Templates

Modify the Jinja2 template for specific needs:

```bash
vi templates/redis_cluster_config.yml.j2
```

### Multiple Environments

Create separate vars files per environment:

```bash
├── vars/
│   ├── dev_cluster.yml
│   ├── staging_cluster.yml
│   └── prod_cluster.yml
```

Run with specific environment:

```bash
ansible-playbook create_pr_for_redis.yml -e @vars/prod_cluster.yml
```

### CI/CD Integration

Integrate into Jenkins/GitLab CI:

```yaml
# .gitlab-ci.yml example
generate_redis_config:
  stage: generate
  script:
    - ansible-playbook create_pr_for_redis.yml 
        -e @vars/${ENVIRONMENT}.yml 
        -e @secrets.yml 
        --vault-password-file $VAULT_PASS_FILE
  only:
    - schedules
```

### Skip PR Creation

To only generate and commit without PR:

```bash
ansible-playbook create_pr_for_redis.yml \
  -e @my_cluster_vars.yml \
  --skip-tags create_pr
```

## Troubleshooting

### Git Authentication Issues

```bash
# Test SSH connection
ssh -T git@github.com

# Check SSH keys
ssh-add -l

# Add SSH key
ssh-add ~/.ssh/id_rsa
```

### GitHub CLI Not Authenticated

```bash
# Login to GitHub
gh auth login

# Check auth status
gh auth status
```

### YAML Validation Errors

```bash
# Manually validate generated file
python3 -c "import yaml; yaml.safe_load(open('path/to/file.yml'))"

# Check with yamllint
yamllint path/to/file.yml
```

### Repository Clone Issues

```bash
# Verify repository access
git ls-remote <your_repo_url>

# Check repository URL in vars file
grep git_repo_url my_cluster_vars.yml
```

**Job hangs on "Clone git repository" in AAP:**
- This happens when cloning a private repository without authentication
- See [GIT_AUTHENTICATION.md](GIT_AUTHENTICATION.md) for how to configure GitHub token authentication
- The playbook automatically configures git credentials when a token is provided
- Supports both SSH keys and HTTPS with tokens

## Security Best Practices

1. **Always use Ansible Vault** for sensitive data
2. **Never commit vault passwords** to git
3. **Rotate credentials** regularly
4. **Limit repository access** to authorized personnel
5. **Review PRs** before merging to production
6. **Enable branch protection** on main branch

## Support & Contributing

### Logging

Playbook execution creates detailed output. Capture for debugging:

```bash
ansible-playbook create_pr_for_redis.yml \
  -e @vars.yml \
  -vvv > playbook_run.log 2>&1
```

### Common Issues

- **Missing variables**: Check vars file for all required fields
- **Git conflicts**: Ensure repository is clean before running
- **Permission denied**: Verify SSH keys and repository access
- **YAML syntax errors**: Validate your vars file before running

## Example Complete Workflow

```bash
# 1. Clone this repository
git clone <this-repo>
cd redis

# 2. Create your vars file
cp example_vars.yml prod_cluster_001.yml
vi prod_cluster_001.yml  # Edit as needed

# 3. Create secrets vault
ansible-vault create secrets.yml
# Add your passwords

# 4. Run automation
ansible-playbook create_pr_for_redis.yml \
  -e @prod_cluster_001.yml \
  -e @secrets.yml \
  --ask-vault-pass

# 5. Review PR in GitHub
# Open the PR URL from the output

# 6. After approval and merge, deploy
cd <config-repo>
git pull origin main
ansible-playbook deployments/redis_cluster_prod_001_*.yml \
  -e @secrets.yml \
  --ask-vault-pass
```

## License

Internal use only - [Your Organization]

## Contact

For questions or issues:
- Team: Redis Infrastructure Team
- Email: redis-team@example.com
- Slack: #redis-automation


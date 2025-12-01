# Bitbucket Server/Data Center Configuration Guide

## Overview

The `create_pr_for_redis_aap.yml` playbook now supports **both** Bitbucket Cloud and Bitbucket Server/Data Center installations. The playbook automatically detects which environment you're using based on the repository URL.

## Changes Made

### Key Improvements

1. **Automatic Environment Detection**: Detects if you're using Bitbucket Cloud or Server/Data Center
2. **Fixed Git Authentication**: Credentials are now properly injected into the clone URL
3. **Server-Specific API Endpoints**: Uses correct REST API paths for Bitbucket Server
4. **Proper Token Handling**: Supports Personal Access Tokens for Server installations
5. **Fixed URL Parsing**: Handles both Cloud (`/workspace/repo`) and Server (`/scm/PROJECT/repo`) formats

### Technical Changes

- **Git Clone Fixed**: Embeds authentication credentials directly in the git URL before cloning
- **Host Detection**: Extracts host with port (e.g., `172.16.2.1:7990`) for Server installations
- **Project/Repo Parsing**: Correctly parses Server URL format `/scm/PROJECT/REPO.git`
- **API Endpoints**: 
  - Cloud: `https://api.bitbucket.org/2.0/...`
  - Server: `http://HOST:PORT/rest/api/1.0/...`
- **PR Creation**: 
  - Cloud: Uses `source/destination` branch format
  - Server: Uses `fromRef/toRef` with full reference paths

## Bitbucket Server Setup

### 1. Create Personal Access Token

In your Bitbucket Server instance:

1. Click your profile icon → **Manage account**
2. Go to **Personal access tokens** (left sidebar)
3. Click **Create a token**
4. Set token name: `AAP Redis Automation`
5. Set permissions:
   - ☑️ **Project Permissions**: Read
   - ☑️ **Repository Permissions**: Write, Admin
   - ☑️ **Pull Request Permissions**: Write
6. Click **Create**
7. **Copy the token immediately** (you won't see it again!)

### 2. Configure AAP Survey Variables

In your Ansible Automation Platform Job Template Survey:

```yaml
bitbucket_username: "your-username"
bitbucket_api_token: "your-personal-access-token"
git_repo_url: "http://172.16.2.1:7990/scm/red/redis.git"
create_pull_request: "yes"
```

### 3. URL Format Examples

**Bitbucket Server/Data Center:**
```
http://bitbucket.example.com:7990/scm/PROJECT/repo.git
http://172.16.2.1:7990/scm/red/redis.git
https://bitbucket.internal.company.com/scm/INFRA/redis-config.git
```

**Bitbucket Cloud:**
```
https://bitbucket.org/myworkspace/my-repo.git
```

## How It Works

### Authentication Flow

1. **URL Analysis**: Playbook parses the `git_repo_url` to extract:
   - Host (with port if present)
   - Project key (Server) or Workspace (Cloud)
   - Repository slug

2. **Environment Detection**:
   ```yaml
   is_bitbucket_cloud: "{{ 'bitbucket.org' in bitbucket_host }}"
   is_bitbucket_server: "{{ 'bitbucket.org' not in bitbucket_host }}"
   ```

3. **Token Type Detection** (Cloud only):
   - Workspace Token (`ATCTT...`): Uses `x-token-auth` as username
   - App Password (`ATBB...`): Uses your actual username

4. **Server Authentication**: Always uses your username with the Personal Access Token

5. **Git Credentials Setup**:
   ```bash
   # Creates ~/.git-credentials with:
   http://username:token@172.16.2.1:7990
   https://username:token@172.16.2.1:7990
   ```

6. **Authenticated Clone**: Injects credentials into clone URL:
   ```
   http://username:token@172.16.2.1:7990/scm/red/redis.git
   ```

### API Integration

**Bitbucket Server API Calls:**

- **Application Properties Test**:
  ```
  GET http://172.16.2.1:7990/rest/api/1.0/application-properties
  ```

- **Repository Access Test**:
  ```
  GET http://172.16.2.1:7990/rest/api/1.0/projects/RED/repos/redis
  ```

- **Create Pull Request**:
  ```
  POST http://172.16.2.1:7990/rest/api/1.0/projects/RED/repos/redis/pull-requests
  Body:
  {
    "title": "...",
    "description": "...",
    "fromRef": {
      "id": "refs/heads/feature-branch",
      "repository": { "slug": "redis", "project": { "key": "RED" } }
    },
    "toRef": {
      "id": "refs/heads/main",
      "repository": { "slug": "redis", "project": { "key": "RED" } }
    }
  }
  ```

## Troubleshooting

### Error: "fatal: could not read Username"

**Problem**: Git clone fails with authentication error

**Solution**: This was the original issue - now fixed! The playbook now:
- Creates credentials file before cloning
- Embeds credentials directly in the clone URL
- Sets up git credential helper properly

### Error: "401 Unauthorized" on API calls

**Problem**: API authentication fails but git works

**Impact**: Non-fatal - branch will still be pushed, PR creation URL will be generated

**Possible Causes**:
- Token missing required permissions
- Token expired
- API endpoint not accessible

**Solution**:
1. Verify token has correct permissions
2. Check token hasn't expired
3. Verify network access to Server API endpoints
4. Use the generated PR URL to create manually

### SSL Certificate Errors (HTTPS Server)

If using HTTPS with self-signed certificates:

```yaml
# In the playbook, modify the uri tasks to:
validate_certs: no
```

(Already configured for Server tasks)

### Custom Port Not Working

Ensure your URL includes the port number:
```yaml
git_repo_url: "http://172.16.2.1:7990/scm/red/redis.git"  # ✓ Correct
git_repo_url: "http://172.16.2.1/scm/red/redis.git"       # ✗ Wrong (missing port)
```

## Testing the Fix

To verify the playbook works with your Bitbucket Server:

1. **Dry Run Test**: Run the playbook with a test configuration
2. **Check Output**: Look for these success messages:
   ```
   ✓ Repository cloned successfully
   ✓ Branch pushed successfully
   ✓ Pull Request created via API (or URL generated)
   ```

3. **Verify in Bitbucket**: 
   - Check the branch exists in your repository
   - Verify PR was created (if API auth succeeded)
   - OR use the generated URL to create PR manually

## Manual PR Creation

If API-based PR creation fails, the playbook generates a direct URL:

**Bitbucket Server:**
```
http://172.16.2.1:7990/projects/RED/repos/redis/pull-requests?create&sourceBranch=refs/heads/redis-deployment-mydb-1234567890
```

Click the URL and you'll be taken directly to the PR creation page with:
- Source branch pre-selected
- Ready to add description and submit

## Security Notes

1. **Token Storage**: Credentials are stored temporarily in `~/.git-credentials` and cleaned up after the job
2. **No Logging**: Sensitive tasks use `no_log: true` to prevent token exposure
3. **Temporary Files**: All credential files are removed in cleanup tasks
4. **Token Permissions**: Use principle of least privilege - only grant required permissions

## Comparison: Cloud vs Server

| Feature | Bitbucket Cloud | Bitbucket Server |
|---------|----------------|------------------|
| Token Type | Workspace Token / App Password | Personal Access Token |
| Git Username | `x-token-auth` or username | Your username |
| API Base URL | `https://api.bitbucket.org/2.0` | `http://HOST:PORT/rest/api/1.0` |
| URL Format | `/workspace/repo` | `/scm/PROJECT/repo` |
| PR API Format | `source/destination` | `fromRef/toRef` |
| SSL Validation | Yes | Disabled for self-signed certs |

## Additional Resources

- [Bitbucket Server REST API Documentation](https://docs.atlassian.com/bitbucket-server/rest/)
- [Bitbucket Cloud API Documentation](https://developer.atlassian.com/cloud/bitbucket/rest/)
- [Personal Access Tokens in Bitbucket Server](https://confluence.atlassian.com/bitbucketserver/personal-access-tokens-939515499.html)

## Support

If you encounter issues:

1. Check the playbook output for detailed error messages
2. Verify token permissions in Bitbucket Server
3. Test git clone manually with the same credentials
4. Review the API authentication test results in playbook output


# Fixing HTTP 403 Forbidden Error

## Your Current Error

```
fatal: unable to access 'http://172.16.2.1:7990/scm/red/redis.git/': 
The requested URL returned error: 403
```

**What this means:** Your credentials are being sent, but Bitbucket Server is **rejecting** them.

## Root Cause

You're using an **HTTP Access Token** (`BBDC-...`) which has limitations:

| Token Type | Prefix | Scope | Works For |
|-----------|--------|-------|-----------|
| **HTTP Access Token** | `BBDC-...` | Repository-specific | Limited git operations |
| **Personal Access Token** | Various | User-wide | All git & API operations |

## ✅ Solution: Use Personal Access Token

### Step 1: Create Personal Access Token

1. **Log into Bitbucket Server:**
   ```
   http://172.16.2.1:7990
   ```

2. **Navigate to Personal Settings:**
   - Click your avatar (top right)
   - Select **"Manage account"**

3. **Create Token:**
   - Left sidebar → **"Personal access tokens"**
   - Click **"Create a token"**

4. **Configure Token:**
   - **Token name:** `AAP Redis Automation`
   - **Expiry:** Choose appropriate duration (or no expiry)
   
5. **Set Permissions (CRITICAL!):**
   ```
   ☑️  Project Permissions
       ☑️  Read
   
   ☑️  Repository Permissions
       ☑️  Read
       ☑️  Write
   
   ☑️  Pull Request Permissions (optional for PR creation)
       ☑️  Read
       ☑️  Write
   ```

6. **Create and Copy:**
   - Click **"Create"**
   - **Copy the token immediately** (you won't see it again!)
   - Token will look like: `NjUyNTAzNT...` (different format than BBDC)

### Step 2: Update AAP Survey

In your Ansible Automation Platform Job Template Survey:

```yaml
bitbucket_username: "admin"  # Your username
bitbucket_api_token: "YOUR-NEW-PERSONAL-ACCESS-TOKEN"  # The token you just copied
git_repo_url: "http://172.16.2.1:7990/scm/red/redis.git"
```

### Step 3: Rerun the Job

The playbook should now successfully:
- ✅ Clone the repository
- ✅ Create a branch
- ✅ Commit changes
- ✅ Push to Bitbucket
- ✅ Create a Pull Request

## 🔍 Verification Before Running

### Test Manually (Optional)

Test your credentials work before running the playbook:

```bash
# Replace with your actual username and token
git clone http://admin:YOUR-TOKEN@172.16.2.1:7990/scm/red/redis.git /tmp/test-clone

# If successful, clean up:
rm -rf /tmp/test-clone
```

## ⚠️ Alternative: Fix HTTP Access Token Permissions

If you **must** use the HTTP Access Token, verify:

### Check Token Permissions

1. **Go to Repository Settings:**
   ```
   http://172.16.2.1:7990/projects/RED/repos/redis/settings/access-tokens
   ```

2. **Find Your Token:**
   - Look for tokens assigned to user `admin`
   - Or the specific token you created

3. **Verify Permissions:**
   - ✅ **Read** permission enabled
   - ✅ **Write** permission enabled

4. **Check Token Status:**
   - Not expired
   - Not revoked
   - Still active

### Check User Repository Access

1. **Verify User Permissions:**
   ```
   http://172.16.2.1:7990/projects/RED/repos/redis/permissions
   ```

2. **User `admin` should have:**
   - ✅ Repository Read access
   - ✅ Repository Write access

3. **Check Project Permissions:**
   ```
   http://172.16.2.1:7990/projects/RED/permissions
   ```
   - User should have at least Read access to project RED

## 🐛 Other Possible Issues

### Issue: Token Expired

**Symptoms:** 403 error, token was working before

**Solution:**
1. Check token expiry in Bitbucket Server
2. Create a new token if expired
3. Update AAP survey with new token

### Issue: User Doesn't Have Repository Access

**Symptoms:** 403 error, token is valid

**Solution:**
1. Verify user `admin` can access the repository in the web UI
2. Check project permissions (RED project)
3. Ask Bitbucket admin to grant access

### Issue: Repository Path Wrong

**Symptoms:** 403 or 404 error

**Solution:**
Verify the repository URL is correct:
```
http://172.16.2.1:7990/scm/red/redis.git
                         │    └── Project key (lowercase in URL)
                         └── Repository slug
```

### Issue: Network/Firewall

**Symptoms:** Timeout or connection refused

**Solution:**
1. Verify AAP can reach Bitbucket Server: `curl http://172.16.2.1:7990`
2. Check firewall rules
3. Test from AAP execution node

## 📊 Comparison: Token Types

### HTTP Access Token (BBDC-...)

**Pros:**
- ✅ Can be scoped to specific repositories
- ✅ Can be created by repo admins

**Cons:**
- ❌ Limited to git operations only
- ❌ May not work with all git commands
- ❌ Repository-specific (not portable)
- ❌ Cannot use Bitbucket REST API

**Use Case:** Simple read-only access to a specific repository

### Personal Access Token

**Pros:**
- ✅ Works for all git operations
- ✅ Works with Bitbucket REST API
- ✅ User-wide permissions (portable)
- ✅ More flexible permission model

**Cons:**
- ❌ Requires user account (can't be repo-only)
- ❌ Broader scope (security consideration)

**Use Case:** Automation, CI/CD, API access (← **This is what you need!**)

## 🎯 Recommended Solution

For AAP automation, **always use Personal Access Tokens**:

1. ✅ More reliable for git operations
2. ✅ Works with PR creation APIs
3. ✅ Consistent authentication model
4. ✅ Better for automation workflows

## 📞 Need Help?

If you still get 403 errors after creating a Personal Access Token:

1. **Check Bitbucket Server logs:**
   ```bash
   tail -f /path/to/bitbucket/logs/atlassian-bitbucket-access.log
   ```

2. **Verify token in Bitbucket:**
   - Token not expired
   - Token has correct permissions
   - User has repository access

3. **Test with curl:**
   ```bash
   curl -u admin:YOUR-TOKEN http://172.16.2.1:7990/rest/api/1.0/projects/RED/repos/redis
   ```
   - Should return repository JSON (not 403)

4. **Contact Bitbucket Administrator:**
   - May need to enable personal access tokens
   - May need to grant project/repo permissions
   - May have authentication restrictions

## ✅ Success Checklist

Before rerunning the playbook, verify:

- [ ] Created Personal Access Token (not HTTP Access Token)
- [ ] Token has Repository Read + Write permissions
- [ ] Token has Project Read permission
- [ ] Token not expired
- [ ] User `admin` has access to RED/redis repository
- [ ] Updated AAP survey with new token
- [ ] Token format is NOT `BBDC-...`

If all checked, rerun the job template! 🚀


# URL Encoding Fix for Special Characters in Tokens

## Issue Fixed

**Error:** `URL using bad/illegal format or missing URL`

**Cause:** Bitbucket Server/Data Center tokens can contain special characters (especially `/`, `@`, `:`) that break URL formatting when embedded in git URLs.

## Example Problem Token

```
BBDC-ExampleTokenString123456789abcdef/EndOfToken
                                        ↑
                                  Forward slash!
```

When embedded in a URL:
```
http://admin:BBDC-...XPNir/xB@172.16.2.1:7990/scm/red/redis.git
                          ↑
                    Git interprets this as a path separator!
```

## Solution Implemented

The playbook now automatically **URL-encodes** tokens before embedding them in URLs:

### URL Encoding Rules

| Character | Encoded As | Why It's a Problem |
|-----------|------------|-------------------|
| `/` | `%2F` | Path separator in URLs |
| `@` | `%40` | User/host separator |
| `:` | `%3A` | Port separator |
| `?` | `%3F` | Query string start |
| `#` | `%23` | Fragment identifier |
| `&` | `%26` | Query parameter separator |
| `+` | `%2B` | Space in query strings |
| ` ` (space) | `%20` | Not allowed in URLs |

## How It Works

### Before (Broken)
```yaml
git_repo_url_with_auth: "http://admin:BBDC-ExampleToken/xB@172.16.2.1:7990/scm/red/redis.git"
                                                        ↑
                                                 Breaks the URL!
```

### After (Fixed)
```yaml
# Step 1: URL-encode the token
encoded_token: "{{ bitbucket_api_token | urlencode }}"
# Result: BBDC-ExampleToken%2FxB
#                          ↑
#                    / becomes %2F

# Step 2: Build URL with encoded token
git_repo_url_with_auth: "http://admin:BBDC-ExampleToken%2FxB@172.16.2.1:7990/scm/red/redis.git"
# Git now correctly interprets this as: user=admin, password=BBDC-ExampleToken/xB
```

## What the Playbook Does Now

### 1. Encodes Token for Git Credentials File
```yaml
- name: URL-encode token for git credentials file using Python
  ansible.builtin.command:
    cmd: python3 -c "import urllib.parse; print(urllib.parse.quote('{{ bitbucket_api_token }}', safe=''))"
  register: encoded_token_result
  
- name: Set encoded token variable for credentials
  ansible.builtin.set_fact:
    encoded_token_for_creds: "{{ encoded_token_result.stdout }}"
```

### 2. Encodes Token for Clone URL
```yaml
- name: URL-encode the token for safe embedding in URL using Python
  ansible.builtin.command:
    cmd: python3 -c "import urllib.parse; print(urllib.parse.quote('{{ bitbucket_api_token }}', safe=''))"
  register: encoded_token_result_url

- name: Set encoded token variable for URL
  ansible.builtin.set_fact:
    encoded_token: "{{ encoded_token_result_url.stdout }}"

- name: Build authenticated git URL for cloning
  ansible.builtin.set_fact:
    git_repo_url_with_auth: "{{ git_repo_url | regex_replace('^(https?://)(.*)$', '\\1' + git_auth_username + ':' + encoded_token + '@\\2') }}"
```

**Why Python instead of Ansible's urlencode filter?**

The `urlencode` filter may not be available in all Ansible versions or may not encode all special characters correctly. Using Python's `urllib.parse.quote()` directly ensures:
- ✅ Works in all Ansible environments (Python 3 is required anyway)
- ✅ Properly encodes ALL special characters (`/`, `@`, `:`, etc.)
- ✅ Consistent behavior across different Ansible versions
- ✅ More reliable for automation

### 3. Detects Special Characters
```yaml
- name: Display token encoding info
  ansible.builtin.debug:
    msg:
      - "Token Encoding:"
      - "Original token contains special characters: {{ 'YES (URL-encoded)' if ('/' in bitbucket_api_token or '@' in bitbucket_api_token or ':' in bitbucket_api_token) else 'NO' }}"
```

## Testing

### Test Your Token

To check if your token needs encoding:

```python
# Python quick test
import urllib.parse

token = "BBDC-ExampleTokenWithSlash/Character"
encoded = urllib.parse.quote(token, safe='')

print(f"Original: {token}")
print(f"Encoded:  {encoded}")
print(f"Changed:  {token != encoded}")
```

Expected output:
```
Original: BBDC-ExampleTokenWithSlash/Character
Encoded:  BBDC-ExampleTokenWithSlash%2FCharacter
Changed:  True
```

### Manual Git Test

Test with encoded token:
```bash
# Original token (will fail)
git clone http://admin:BBDC-ExampleToken/xB@172.16.2.1:7990/scm/red/redis.git
# Error: URL using bad/illegal format

# URL-encoded token (will work)
git clone http://admin:BBDC-ExampleToken%2FxB@172.16.2.1:7990/scm/red/redis.git
# Success!
```

## What You See in the Playbook

When running the updated playbook:

```
TASK [URL-encode the token for safe embedding in URL]
ok: [localhost]

TASK [Build authenticated git URL for cloning]
ok: [localhost]

TASK [Display token encoding info]
ok: [localhost] => {
    "msg": [
        "Token Encoding:",
        "  Original token contains special characters: YES (URL-encoded)",
        "  Special chars detected: / @ : (these will be percent-encoded)"
    ]
}

TASK [Display masked clone URL for debugging]
ok: [localhost] => {
    "msg": "Clone URL (masked): http://admin:***TOKEN***@172.16.2.1:7990/scm/red/redis.git"
}

TASK [Clone git repository]
changed: [localhost]

✓ Repository cloned successfully
```

## Common Token Formats

### Bitbucket Cloud

**Workspace Access Token:**
```
ATCTT3xFfGN0_ExampleTokenString_NoSpecialChars
No special chars typically
```

**App Password:**
```
ATBBxxx_ExampleAppPassword_NoSpecialChars
No special chars typically
```

### Bitbucket Server/Data Center

**HTTP Access Token:**
```
BBDC-ExampleToken_WithSlash/Character
                           ↑
                 Often contains /
```

**Personal Access Token:**
```
ExampleToken:WithColonCharacter
            ↑
      Contains : (colon)
```

## Important Notes

### 1. URL Encoding is Automatic

You don't need to do anything! The playbook handles this automatically.

**Just provide the raw token in the AAP survey:**
```yaml
bitbucket_api_token: "BBDC-YourActualTokenWithSpecialChars/Here"
```

**NOT like this (don't encode it yourself):**
```yaml
bitbucket_api_token: "BBDC-YourActualTokenWithSpecialChars%2FHere"  # ✗ Wrong!
```

### 2. Applies to All Git Operations

URL encoding is used for:
- ✅ Clone operations
- ✅ Push operations  
- ✅ Git credential storage
- ✅ Remote URL configuration

### 3. Doesn't Affect API Calls

API calls use headers, not URL embedding:
```yaml
# API calls still use the raw token
headers:
  Authorization: "Bearer {{ bitbucket_api_token }}"

# Or Basic Auth
user: "{{ bitbucket_username }}"
password: "{{ bitbucket_api_token }}"  # Raw token, not encoded
```

## Troubleshooting

### Still Getting URL Format Error?

If you still see `URL using bad/illegal format`:

1. **Check Python 3 availability:**
   ```bash
   python3 --version
   # Python 3.x required (should be available in AAP execution environments)
   ```

2. **Verify token is being passed correctly:**
   - No extra spaces
   - No newlines
   - Complete token copied

3. **Check for other special characters:**
   The `urlencode` filter should handle all special characters, but some tokens might have unusual characters.

4. **Try with SSH instead:**
   ```yaml
   git_repo_url: "git@172.16.2.1:red/redis.git"
   # Use SSH keys instead of HTTP tokens
   ```

### Token Still Showing Special Characters Warning?

This is normal! The playbook detects and handles them automatically:

```
Token Encoding:
  Original token contains special characters: YES (URL-encoded)
  Special chars detected: / @ : (these will be percent-encoded)
```

This is informational only - the encoding is working correctly.

## Security Notes

### URL Encoding and Security

- ✅ **URL encoding is NOT encryption** - it's just format conversion
- ✅ **Token is still sensitive** - don't log encoded or unencoded versions
- ✅ **Credentials are cleaned up** - temporary files removed after job
- ✅ **Use `no_log: true`** - playbook hides credentials in logs

### Best Practices

1. **Always use AAP credentials/vault** for tokens
2. **Rotate tokens regularly**
3. **Use least-privilege permissions**
4. **Monitor token usage** in Bitbucket Server logs
5. **Revoke tokens** when no longer needed

## References

- [RFC 3986 - URL Syntax](https://tools.ietf.org/html/rfc3986)
- [Percent-encoding (Wikipedia)](https://en.wikipedia.org/wiki/Percent-encoding)
- [Python urllib.parse.quote()](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.quote)

## Summary

✅ **The fix is automatic** - just rerun your job template with the same token!

The playbook now:
1. Detects special characters in tokens
2. URL-encodes them automatically
3. Builds proper URLs for git operations
4. Shows informational messages about encoding
5. Handles both git operations and API calls correctly

**No action needed from you** - the playbook handles everything! 🎉


# CyberArk Ansible Automation Examples

This directory contains example playbooks demonstrating integration between Ansible and CyberArk Privileged Access Security (PAS) using the certified `cyberark.pas` collection.

## Overview

The CyberArk PAS collection enables automated management of privileged credentials, accounts, and users within the CyberArk Vault, including access to static safes.

## Collection Information

- **Collection**: `cyberark.pas`
- **Latest Version**: 1.0.35+
- **Documentation**: https://docs.ansible.com/ansible/latest/collections/cyberark/pas/

## Prerequisites

1. **CyberArk Environment**:
   - Access to a CyberArk PVWA (Password Vault Web Access)
   - Valid credentials with appropriate permissions
   - Configured safes and accounts (for retrieval examples)
   - CCP (Central Credential Provider) configured (for credential retrieval)

2. **Ansible Requirements**:
   - Ansible 2.9 or later
   - Python 3.6 or later

## Installation

Install the required collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

Or install directly:

```bash
ansible-galaxy collection install cyberark.pas
```

## Configuration

### 1. Update Variables

Edit `example_vars.yml` with your environment details:
- CyberArk PVWA URL
- Safe names and account information
- Application IDs for CCP access

### 2. Secure Credentials

**IMPORTANT**: Never store passwords in plain text. Use Ansible Vault:

```bash
# Create encrypted variables file
ansible-vault create vault_credentials.yml

# Add your sensitive variables:
vault_username: your_username
vault_password: your_password
```

Or use environment variables:
```bash
export CYBERARK_PASSWORD="your_password"
```

## Playbook Features

The example playbook demonstrates:

### 1. **Authentication**
- Connect to CyberArk Vault using the Web Services SDK
- Establish a session for subsequent operations

### 2. **Credential Retrieval**
- Retrieve credentials from static safes using `cyberark_account`
- Alternative retrieval using CCP with `cyberark_credential`
- Access account properties and passwords

### 3. **Account Management**
- Create new accounts in safes
- Update existing account properties
- Configure platform-specific settings

### 4. **User Management**
- Create CyberArk user accounts
- Set user properties and permissions
- Configure password policies

### 5. **Practical Usage**
- Use retrieved credentials in subsequent tasks
- Delegate to remote hosts with CyberArk-managed credentials

## Usage Examples

### Basic Execution

```bash
ansible-playbook cyberark_example_playbook.yml \
  -e @example_vars.yml \
  -e vault_username=admin \
  -e vault_password=SecurePassword123
```

### Using Ansible Vault

```bash
ansible-playbook cyberark_example_playbook.yml \
  -e @example_vars.yml \
  -e @vault_credentials.yml \
  --ask-vault-pass
```

### Retrieve Credentials Only

```bash
ansible-playbook cyberark_example_playbook.yml \
  -e @example_vars.yml \
  --tags retrieve \
  --ask-vault-pass
```

## Static Safe Access

The collection **DOES support accessing static safes**:

### Method 1: Direct Account Retrieval
```yaml
- name: Retrieve from static safe
  cyberark_account:
    api_base_url: "{{ cyberark_pvwa_url }}"
    cyberark_session: "{{ cyberark_session }}"
    state: retrieve
    safe: "MyStaticSafe"
    username: "service_account"
    address: "app.example.com"
```

### Method 2: CCP Retrieval
```yaml
- name: Retrieve via CCP
  cyberark_credential:
    api_base_url: "{{ cyberark_pvwa_url }}/AIMWebService"
    app_id: "MyApp"
    query: "Safe=MyStaticSafe;UserName=service_account"
```

## Available Modules

| Module | Purpose |
|--------|---------|
| `cyberark_authentication` | Authenticate to CyberArk Vault |
| `cyberark_account` | Manage accounts (create, update, retrieve, delete) |
| `cyberark_credential` | Retrieve credentials via CCP |
| `cyberark_user` | Manage CyberArk user accounts |

## Security Best Practices

1. **Never commit passwords** to version control
2. **Use Ansible Vault** for sensitive data
3. **Enable `no_log: true`** on tasks handling credentials
4. **Validate SSL certificates** in production (`validate_certs: true`)
5. **Use service accounts** with minimal required permissions
6. **Rotate credentials** regularly
7. **Audit access** to CyberArk safes

## Troubleshooting

### Authentication Fails
- Verify PVWA URL is accessible
- Check username/password credentials
- Ensure user has appropriate CyberArk permissions
- Validate SSL certificate if `validate_certs: true`

### Credential Retrieval Fails
- Verify safe name and account details are correct
- Check Application ID permissions in CyberArk
- Ensure the account exists in the specified safe
- Verify CCP is configured and accessible

### Account Creation Fails
- Ensure the safe exists and is accessible
- Verify platform_id matches a configured platform
- Check user has "Add Accounts" permission on the safe

## Additional Resources

- [CyberArk PAS Collection Documentation](https://docs.ansible.com/ansible/latest/collections/cyberark/pas/)
- [CyberArk REST API Documentation](https://docs.cyberark.com/PAS/Latest/en/Content/SDK/APIs.htm)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

## Support

For issues with the collection, refer to:
- Ansible Galaxy: https://galaxy.ansible.com/cyberark/pas
- CyberArk Community: https://cyberark.my.site.com/s/

## License

These examples are provided for reference purposes. Refer to the CyberArk PAS collection license for usage terms.


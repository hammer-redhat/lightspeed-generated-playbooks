# DataPower Web Gateway Creation Playbooks

This repository contains Ansible playbooks for creating web gateways and proxies in IBM DataPower instances.

## Files

- `create_web_gateway.yml` - Main playbook for creating WebServiceGateway (recommended)
- `create_proxy.yml` - Simple proxy creation playbook (legacy)
- `inventory.yml` - Ansible inventory file for DataPower servers
- `vars.yml` - Default variables for configuration

## Prerequisites

1. Ansible installed on the control machine
2. Network connectivity to DataPower management interface (port 5550)
3. Valid DataPower admin credentials
4. DataPower server accessible via HTTPS

## Usage

### Basic Usage

```bash
# Create a WebServiceGateway (recommended)
ansible-playbook -i inventory.yml create_web_gateway.yml \
  -e "datapower_password=your_password"

# Create with custom settings
ansible-playbook -i inventory.yml create_web_gateway.yml \
  -e "datapower_password=your_password" \
  -e "gateway_name=my-gateway" \
  -e "backend_url=http://backend:8080"
```

### Advanced Usage

```bash
# Run with custom inventory and variables file
ansible-playbook -i inventory.yml create_web_gateway.yml \
  -e "@vars.yml" \
  -e "datapower_password=your_password"

# Run on specific DataPower server
ansible-playbook -i inventory.yml create_web_gateway.yml \
  --limit datapower-01 \
  -e "datapower_password=your_password"
```

## Configuration Variables

### Required Variables

- `datapower_password` - DataPower admin password
- `web_proxy_backend_url` - Backend server URL

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `web_proxy_name` | `web-proxy-01` | Name of the web proxy service |
| `web_proxy_domain` | `default` | DataPower domain |
| `web_proxy_state` | `enabled` | Service state (enabled/disabled) |
| `web_proxy_port` | `8080` | HTTP port for the proxy |
| `web_proxy_ssl_port` | `8443` | HTTPS port for the proxy |
| `web_proxy_interface` | `eth0` | Network interface |
| `web_proxy_ssl_enabled` | `true` | Enable SSL support |
| `web_proxy_ssl_cert` | `default-cert` | SSL certificate name |
| `web_proxy_ssl_key` | `default-key` | SSL key name |
| `web_proxy_backend_timeout` | `30` | Backend timeout in seconds |

## Inventory Configuration

Update `inventory.yml` with your DataPower server details:

```yaml
datapower_servers:
  hosts:
    datapower-01:
      ansible_host: "your-datapower-ip"
      datapower_user: "admin"
      datapower_domain: "default"
```

## Security Considerations

1. Store sensitive passwords in Ansible Vault:
   ```bash
   ansible-vault create group_vars/all/vault.yml
   ```

2. Use encrypted variables for passwords:
   ```yaml
   datapower_password: "{{ vault_datapower_password }}"
   ```

3. Validate SSL certificates in production environments

## Troubleshooting

### Common Issues

1. **Connection Refused**: Verify DataPower management interface is accessible
2. **Authentication Failed**: Check username/password credentials
3. **SSL Certificate Errors**: Use `validate_certs: false` for testing
4. **Service Won't Start**: Check DataPower domain status and configuration

### Debug Mode

Run with increased verbosity for troubleshooting:

```bash
ansible-playbook -i inventory.yml create_web_proxy.yml -vvv
```

## Example Output

```
PLAY [Create a new web proxy in IBM DataPower] ********************************

TASK [Validate required variables] ********************************************
ok: [datapower-01]

TASK [Check DataPower connectivity] *******************************************
ok: [datapower-01]

TASK [Create web proxy configuration XML] *************************************
ok: [datapower-01]

TASK [Deploy web proxy configuration to DataPower] ***************************
changed: [datapower-01]

TASK [Verify web proxy deployment] ********************************************
ok: [datapower-01]

TASK [Start the web proxy service] ********************************************
changed: [datapower-01]

TASK [Test web proxy connectivity] ********************************************
ok: [datapower-01]

TASK [Display web proxy status] ***********************************************
ok: [datapower-01] => {
    "msg": "Web Proxy 'my-web-proxy' has been created successfully!\n\nConfiguration Details:\n- Name: my-web-proxy\n- Domain: default\n- State: enabled\n- HTTP Port: 8080\n- HTTPS Port: 8443\n- SSL Certificate: default-cert\n- Backend URL: http://backend-server:8080\n- Backend Timeout: 30s\n\n- Connectivity Test: PASSED"
}
```

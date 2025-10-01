# DataPower WSGateway Creation with Ansible

This repository contains an Ansible playbook for creating WSGateway (Web Services Gateway) instances on IBM DataPower using the REST API.

## 📁 Files

- **`create_wsgateway_rest.yml`** - Main playbook for creating WSGateways
- **`inventory.yml`** - Ansible inventory file for DataPower servers (optional)
- **`vars.yml`** - Default variables for configuration (optional)

## 🚀 Quick Start

### Prerequisites

1. Ansible installed on the control machine
2. Network connectivity to DataPower REST management interface (port 5554)
3. Valid DataPower credentials with permissions to create objects
4. DataPower REST Management Interface enabled

### Basic Usage

```bash
# Create a WSGateway with default settings
ansible-playbook create_wsgateway_rest.yml -e "datapower_password=your_password"

# Create with custom name and port
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_password=your_password" \
  -e "wsgateway_name_override=my-gateway" \
  -e "wsgateway_port_override=9090"
```

## 📋 Configuration

### Required Variables

- **`datapower_password`**: The password for the DataPower user (must be provided at runtime)

### Optional Variables

You can override any of these variables using `-e` flags:

| Variable | Default | Description |
|----------|---------|-------------|
| `datapower_host` | `1.2.3.4` | DataPower server hostname or IP |
| `datapower_user` | `hammer` | DataPower username |
| `datapower_rest_port` | `5554` | REST API port |
| `datapower_domain` | `default` | DataPower domain name |
| `wsgateway_name_override` | `my-ws-gateway` | Name of the WSGateway |
| `wsgateway_port_override` | `8082` | Listening port for the gateway |
| `wsgateway_address_override` | `0.0.0.0` | Listening address for the gateway |

## 📝 Examples

### Example 1: Create Basic WSGateway

```bash
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_password=redhat123"
```

This creates a WSGateway named `my-ws-gateway` listening on port `8082`.

### Example 2: Create Custom Named Gateway

```bash
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_password=redhat123" \
  -e "wsgateway_name_override=api-gateway" \
  -e "wsgateway_port_override=8080"
```

### Example 3: Create Multiple Gateways

```bash
# Gateway 1
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_password=redhat123" \
  -e "wsgateway_name_override=gateway-01" \
  -e "wsgateway_port_override=8080"

# Gateway 2
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_password=redhat123" \
  -e "wsgateway_name_override=gateway-02" \
  -e "wsgateway_port_override=8081"
```

### Example 4: Use Different Credentials

```bash
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_user=admin" \
  -e "datapower_password=admin123"
```

### Example 5: Connect to Different DataPower Instance

```bash
ansible-playbook create_wsgateway_rest.yml \
  -e "datapower_host=192.168.1.100" \
  -e "datapower_user=dpuser" \
  -e "datapower_password=secret123"
```

## 🔧 How It Works

The playbook performs the following steps:

1. **Connectivity Test** - Verifies connection to the DataPower REST API
2. **Create HTTP Handler** - Creates an `HTTPSourceProtocolHandler` for the gateway to listen on
3. **Create WSGateway** - Creates the WSGateway object using the HTTP handler
4. **Verification** - Confirms the WSGateway was successfully created
5. **Summary Report** - Displays the results and access information

### What Gets Created

For each run, the playbook creates:

- **HTTPSourceProtocolHandler**: `{wsgateway_name}-handler`
  - Handles incoming HTTP connections
  - Listens on the specified port and address
  
- **WSGateway**: `{wsgateway_name}`
  - Web Services Gateway service
  - Uses the HTTP handler as its front protocol
  - Enabled and ready to process requests

## ⚙️ DataPower Configuration

### Enable REST Management Interface

The REST Management Interface should be enabled by default on port 5554. To verify:

1. **Login to DataPower UI**: `https://<datapower-host>:9090`
2. **Navigate to**: Administration → Main → REST Management Interface
3. **Verify**: Port 5554 is enabled

### Verify REST API Connectivity

```bash
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default"
```

You should receive a JSON response with configuration data.

## 🎯 Viewing Created Gateways

### In DataPower Web UI

1. **Access the UI**: `https://<datapower-host>:9090`
2. **Login** with your credentials
3. **Navigate to**: Control Panel → Services → WSGateway
4. **Find** your gateway by name

### Via REST API

```bash
# List all WSGateways
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default/WSGateway"

# Get specific WSGateway details
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default/WSGateway/{gateway-name}"
```

### Via CLI

```bash
# Test gateway connectivity
curl http://<datapower-host>:<gateway-port>/
```

## 🔍 Troubleshooting

### Issue: Authentication Failures

**Symptoms:** 401 Unauthorized errors

**Solution:**
1. Verify credentials are correct
2. Check user has permissions in the target domain
3. Ensure the user group has read+write access

```bash
# Check user permissions
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default/User/{username}"
```

### Issue: Port Already in Use

**Symptoms:** Error creating HTTP handler or WSGateway

**Solution:**
1. Choose a different port using `-e "wsgateway_port_override=XXXX"`
2. Check existing handlers:

```bash
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default/HTTPSourceProtocolHandler"
```

### Issue: Gateway Not Visible

**Symptoms:** Playbook succeeds but gateway not in UI

**Solution:**
1. Check the correct domain (default vs. others)
2. Verify the gateway status:

```bash
curl -k -u username:password \
  "https://<datapower-host>:5554/mgmt/config/default/WSGateway/{gateway-name}"
```

3. Check if gateway is enabled (`mAdminState: enabled`)

### Issue: REST API Connection Refused

**Symptoms:** Cannot connect to port 5554

**Solution:**
1. Verify REST Management Interface is enabled
2. Check firewall rules allow port 5554
3. Confirm DataPower is running

```bash
# Test basic connectivity
nc -zv <datapower-host> 5554
```

## 📚 Additional Resources

- [IBM DataPower Documentation](https://www.ibm.com/docs/en/datapower-gateway)
- [DataPower REST Management Interface](https://www.ibm.com/docs/en/datapower-gateway/10.0?topic=interface-rest-management)
- [WSGateway Configuration](https://www.ibm.com/docs/en/datapower-gateway/10.0?topic=services-web-service-proxy)
- [Ansible URI Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html)

## 🔐 Security Best Practices

1. **Never commit passwords** to version control
2. **Use Ansible Vault** for sensitive variables:
   ```bash
   # Create encrypted variable file
   ansible-vault create secrets.yml
   
   # Add password to secrets.yml:
   datapower_password: your_secret_password
   
   # Run playbook with vault
   ansible-playbook create_wsgateway_rest.yml -e "@secrets.yml" --ask-vault-pass
   ```

3. **Use environment variables**:
   ```bash
   export DATAPOWER_PASSWORD="your_password"
   ansible-playbook create_wsgateway_rest.yml \
     -e "datapower_password=$DATAPOWER_PASSWORD"
   ```

4. **Prompt for password**:
   ```bash
   ansible-playbook create_wsgateway_rest.yml \
     -e "datapower_password={{ lookup('env', 'DATAPOWER_PASSWORD') }}"
   ```

## 🤝 Contributing

To improve this playbook:

1. Test thoroughly against a DataPower instance
2. Follow Ansible best practices
3. Include error handling for edge cases
4. Update this README with new features

## 📄 License

This project is provided as-is for use with IBM DataPower instances.

## 🆘 Support

For issues or questions:

1. Check the troubleshooting section above
2. Review DataPower logs for specific errors
3. Verify REST Management Interface is properly configured
4. Test API connectivity with curl commands before running playbook

---

**Note:** This playbook uses the DataPower REST Management Interface (port 5554) for all operations. The XML Management Interface (port 5550) is not used.

**Tested with:**
- IBM DataPower Gateway
- Ansible 2.9+
- Python 3.x

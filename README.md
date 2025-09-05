# Lightspeed Generated Playbooks

Ansible automation playbooks for accelerating infrastructure and application provisioning across multiple platforms and services.  
This repository is designed to centralize and standardize automation for various enterprise use cases, enabling faster deployments and consistent configurations.

---

## 📌 Overview

The **lightspeed-generated-playbooks** repository houses Ansible playbooks automatically generated and optimized for different platforms and services. These playbooks aim to:

- Simplify provisioning and configuration.
- Ensure consistency across environments.
- Accelerate infrastructure and application deployment.
- Provide reusable automation components for teams.

---

## 🚀 Supported Use Cases

This repository provides playbooks for the following platforms and services:

- **Databases**
  - Redis
  - MySQL
  - MongoDB
  - Oracle Exadata
  - Teradata
  - Neo4J
- **Messaging & Streaming**
  - Kafka
  - MQ / RDQM
- **Application & API Platforms**
  - Apigee API Enablement
  - Informatica PowerCenter / PowerExchange
  - Integration Product Team (BIZ, EAI, ESI, IIB, WPI)
  - SSIS and SSRS Server Configuration
  - ServiceMesh
- **Security & Identity**
  - CyberArk
  - Non-Human Service ID Management
  - Test ID Non-Human Account
- **Networking & Load Balancing**
  - F5 Load Balancers
  - Broadcom CA7
- **Monitoring & Observability**
  - Dynatrace Monitoring
- **File Transfer & Automation**
  - Electronic File Transfer (EFX) Automation
  - Sterling File Gateway (SFG)
- **Storage & Compute**
  - NAS Configuration
  - Compute Resize
- **Other Services**
  - RDSH/HAI (Horizon)
  - Informatica PowerCenter / Power Exchange

---

## 🛠️ Prerequisites

Before using these playbooks, ensure you have the following:

- **Ansible** >= 2.15  
- Access to target infrastructure and credentials where applicable.
- Required collections and roles installed (documented in each playbook's README).
- `oc` CLI installed if working with OpenShift-related resources.
- Proper credentials for APIs and databases.

---

## 📂 Repository Structure

```bash
lightspeed-generated-playbooks/
├── redis/
├── mysql/
├── f5/
├── cyberark/
├── kafka/
├── ssis-ssrs/
├── broadcom-ca7/
├── apigee/
├── mq-rdqm/
├── mongodb/
├── dynatrace/
├── nas/
├── compute-resize/
├── service-ids/
├── oracle-exadata/
├── rdsh-hai/
├── integration-products/
├── informatica/
├── teradata/
├── neo4j/
├── servicemesh/
├── efx-sfg/
└── README.md

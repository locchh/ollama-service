# Setup Machine

Cloud Provider: Azure (0.4400 USD/hr)

## Machine Configuration

### 1. Basics

- Resource group: `ollama-service`
- Virtual machine name: `vm-ollama-service`
- Region: (US) East US
- Availability zone: Zone 1
- Security type: Standard

### 2. Hardware

- OS: Ubuntu Server 22.04 LTS - x64 Gen2
- VM architecture: x64
- Type: Standard_D8s_v3
- RAM: 32GB
- CPU: 8 vCPUs
- Disk (SSD): 256 GB

### 3. Network

- Virtual network: (New) vnet-centralus (ollama-service)
- Subnet: (New) snet-centralus-1 (172.16.0.0 - 172.16.0.255 (256 addresses))
- NIC network security group: Basic
- Delete public IP and NIC when VM is deleted: Yes
- Load balancing options: None

### 4. Monitor
- Health monitor configuration: Type: Application health extension
Protocol: HTTP
Port number: 80
Path: /
Unhealthy threshold: 5
Interval (sec): 5


## Report

```
Basics
Subscription: Azure subscription 1
Resource group: (new) ollama-service
Virtual machine name: vm-ollama-service
Region: Central US
Availability options: Availability zone
Zone options: Self-selected zone
Availability zone: 1
Security type: Standard
Image: Ubuntu Server 22.04 LTS - Gen2
VM architecture: x64
Size: Standard D8s v3 (8 vcpus, 32 GiB memory)
Enable Hibernation: No
Authentication type: SSH public key
Username: azureuser
SSH Key format: RSA
Key pair name: vm-ollama-service_key
Public inbound ports: SSH, HTTP, HTTPS
Azure Spot: No

Disks
OS disk size: 256 GiB
OS disk type: Standard SSD LRS
Use managed disks: Yes
Delete OS disk with VM: Enabled
Ephemeral OS disk: No

Networking
Virtual network: vnet-centralus
Subnet: snet-centralus-1
Public IP: (new) vm-ollama-service-ip
Accelerated networking: Off
Place this virtual machine behind an existing load balancing solution?: No
Delete public IP and NIC when VM is deleted: Enabled


Management
Microsoft Defender for Cloud: Basic (free)
System assigned managed identity: Off
Login with Microsoft Entra ID: Off
Auto-shutdown: Off
Backup: Disabled
Enable periodic assessment: Off
Enable hotpatch: Off
Patch orchestration options: Azure-orchestrated patching (preview): patches will be installed by Azure
Reboot setting: Reboot if required


Monitoring
Alerts: Off
Boot diagnostics: On
Enable OS guest diagnostics: Off
Enable application health monitoring: On


Advanced
Extensions: None
VM applications: None
Cloud init: No
User data: No
Disk controller type: SCSI
Proximity placement group: None
Capacity reservation group: None
```
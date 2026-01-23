# Installation Guide

## Prerequisites

Before installing Guardian Pro, ensure you have:

- **Cloud Infrastructure**: AWS, Azure, or Google Cloud account with provisioning permissions
- **Network Access**: Ability to whitelist external IPs and configure firewall rules
- **PACS Integration**: Access to your Picture Archiving and Communication System
- **Radiologist List**: Current roster of radiologists for enrollment
- **Technical Support**: IT administrator familiar with cloud infrastructure, networking, and database administration
- **PostgreSQL Database**: External or cloud-hosted PostgreSQL instance

## Installation Checklist

Download our [Installation Preparation Spreadsheet](https://docs.zauronlabs.com/guardian-pro/installation-checklist.xlsx) to track your setup progress. This spreadsheet includes placeholders for all required configuration details.

### Sample Configuration Spreadsheet

| Component | Username | Password | Endpoint/Host | Port | Complete (Yes/No) | Notes |
|-----------|----------|----------|---------------|------|-------------------|-------|
| VM SSH Access | zauron | N/A | [VM Public IP] | 22 | No | SSH key required |
| PostgreSQL Database | [db_username] | [db_password] | [db_host] | 5432 | No | Version 13+ required |
| Guardian PACS Listener | N/A | N/A | [VM Public IP] | 4000 | No | DICOM listener port |
| Guardian AI Dashboard | N/A | N/A | [VM Public IP] | 8090 | No | Web dashboard access |
| Guardian Viewer | N/A | N/A | [VM Public IP] | 80/443 | No | HTTPS access required |

**Instructions:**
1. Download the spreadsheet template from the link above
2. Fill in the bracketed placeholder values with your actual configuration
3. Mark "Complete" as Yes/No as you provision each component
4. Use the Notes column for any additional configuration details
5. Send the completed spreadsheet to your Zauron representative before installation

## Deployment Options

Guardian Pro offers two deployment models to suit different organizational needs and security requirements.

## Option 1: On-Premises Deployment

### Infrastructure Requirements

#### Cloud Space Provisioning
1. **Choose Cloud Provider**: AWS, Azure, or Google Cloud Platform
2. **VM Specifications**:
   - **Required**: Ubuntu VM with 12 vCPU cores and 32GB RAM
   - **Storage**: Minimum 200GB SSD storage
   - **SSH Access**: SSH key provided to username `zauron` with sudo privileges

#### Database Requirements
- **PostgreSQL Database**: Version 13 or later
- **Connection Details**: Username and password for database access
- **Network Access**: Database must be accessible from the Guardian Pro VM

#### Network and Security Requirements
- **SSH Key**: Provide SSH private key for VM access
- **Firewall Ports**: Open the following incoming ports:
  - `22`: SSH access
  - `80/443`: Guardian Viewer (HTTP/HTTPS)
  - `4000`: Guardian PACS listener
  - `8090`: Guardian AI Governance dashboard

#### System Requirements
- **Operating System**: Ubuntu 20.04 LTS or later
- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 2.0 or later
- **SSL Certificate**: Valid certificate for HTTPS endpoints

### Installation Steps

#### 1. Provision Cloud Infrastructure

**VM Specifications:**
- Ubuntu 20.04 LTS or later
- 12 vCPU cores, 32GB RAM
- 200GB SSD storage minimum
- SSH key access for user `zauron` with sudo privileges

```bash
# Example AWS EC2 instance creation (t3.2xlarge = 8 vCPU, 32GB RAM - upgrade to c5.4xlarge for 16 vCPU if needed)
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --count 1 \
  --instance-type c5.4xlarge \
  --key-name zauron-ssh-key \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --user-data '#!/bin/bash
    useradd -m -s /bin/bash zauron
    usermod -aG sudo zauron
    mkdir -p /home/zauron/.ssh
    echo "ssh-rsa YOUR_PUBLIC_KEY_HERE" >> /home/zauron/.ssh/authorized_keys
    chown -R zauron:zauron /home/zauron/.ssh
    chmod 600 /home/zauron/.ssh/authorized_keys
  '
```

**Required Ports to Open:**
- `22`: SSH (for zauron user)
- `80/443`: Guardian Viewer (HTTP/HTTPS)
- `4000`: Guardian PACS listener
- `8090`: Guardian AI Governance dashboard

#### 2. Configure Database Access

**PostgreSQL Setup:**
1. Ensure PostgreSQL 13+ is running and accessible from the VM
2. Create a database user with appropriate permissions
3. Note the connection details for the installation checklist

**Required Information:**
- Database host/IP address
- Database port (default: 5432)
- Database name
- Username with read/write permissions
- Password for the database user

#### 4. Configure PACS Integration

**Whitelist Zauron DICOM Node:**
- IP Address: [VM Public IP Address]
- AE Title: Zauron_Query
- Port: 4000 (Guardian PACS listener)

**Configuration Steps:**
1. Access your PACS administration console
2. Navigate to network/remote nodes configuration
3. Add new DICOM node with provided credentials
4. Test connectivity using DICOM echo

#### 5. Configure Reporting System Integration

**Whitelist Zauron Reader Node:**
- IP Address: [VM Public IP Address]
- Integration Method: HL7 or direct API (depending on your RIS)
- Authentication: Mutual TLS or API key

#### 6. Install Guardian Pro Software

```bash
# Download installation package
wget https://downloads.zauron.com/guardian-pro/installer.sh
chmod +x installer.sh

# Run installer
sudo ./installer.sh --config on-prem
```

#### 7. Configure Radiologist Enrollment

Prepare a CSV file with radiologist information:
```csv
email,name,department,specialty
dr.smith@hospital.com,Dr. John Smith,Radiology,MSK
dr.jones@hospital.com,Dr. Sarah Jones,Radiology,Neuro
```

Upload the file through the web interface or provide to your Zauron representative.

### Post-Installation Verification

1. **Health Check**: Access the web dashboard and verify system status
2. **DICOM Connectivity**: Test study retrieval from PACS
3. **Email Delivery**: Verify radiologists receive test notifications
4. **API Integration**: Confirm reporting system receives peer review results

## Option 2: Cloud Deployment

### Infrastructure Requirements

#### VPN Configuration
- **Site-to-Site VPN**: Between your network and Zauron's cloud environment
- **Supported Protocols**: IPsec, OpenVPN, or AWS VPC peering
- **Bandwidth**: Minimum 100Mbps dedicated circuit

#### Data Security
- **Encryption**: All data encrypted in transit and at rest
- **Anonymization**: DICOM studies automatically anonymized before transmission
- **Compliance**: HIPAA, GDPR, and SOC 2 compliant infrastructure

### Installation Steps

#### 1. Network Preparation

**Establish VPN Connection:**
```bash
# Example OpenVPN configuration
client
dev tun
proto udp
remote vpn.zauron.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
```

#### 2. Configure DICOM Forwarding

**Whitelist Zauron DICOM Receiver:**
- IP Address: [Cloud endpoint provided by Zauron]
- AE Title: Zauron_Cloud
- Port: 104

**Routing Configuration:**
1. Configure your PACS to forward studies to Zauron's cloud endpoint
2. Set up conditional forwarding rules (e.g., specific modalities or departments)
3. Test DICOM connectivity through VPN tunnel

#### 3. Set Up Radiologist Access

**Email Configuration:**
- Provide list of radiologist email addresses
- Configure email domains for whitelisting
- Set up secure link distribution

**User Training:**
- Most radiologists require no training (familiar with PACS workflows)
- Optional: 15-minute onboarding session available

#### 4. Activate Service

```bash
# Verify VPN connectivity
ping vpn.zauron.com

# Test DICOM forwarding
# (Use your PACS testing tools)

# Activate Guardian Pro
curl -X POST https://api.zauron.com/activate \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"organization_id": "your-org-id"}'
```

### Post-Installation Verification

1. **VPN Connectivity**: Confirm stable connection to Zauron cloud
2. **Data Flow**: Verify anonymized studies reach cloud environment
3. **Peer Review Workflow**: Test complete review cycle from study to completion
4. **Reporting Integration**: Ensure results flow back to your reporting system

## Common Configuration Issues

### DICOM Connectivity Problems
- **Symptom**: Studies not appearing in Guardian Pro
- **Solution**: Verify AE titles, ports, and firewall rules
- **Debug**: Use DICOM verification tools like `dcmtk` or `pynetdicom`

### VPN Connection Issues
- **Symptom**: Intermittent connectivity or slow performance
- **Solution**: Check bandwidth, latency, and VPN configuration
- **Debug**: Run continuous ping tests and monitor packet loss

### Email Delivery Problems
- **Symptom**: Radiologists not receiving review notifications
- **Solution**: Check spam filters, email domains, and SMTP configuration
- **Debug**: Send test emails and verify delivery logs

## Security Considerations

### Data Protection
- **Encryption**: All data encrypted using AES-256
- **Access Control**: Role-based access with multi-factor authentication
- **Audit Logging**: Complete audit trails for all system activities

### Network Security
- **Firewall Rules**: Minimal required ports open
- **Intrusion Detection**: Continuous monitoring for security threats
- **Regular Updates**: Automated security patching and updates

## Support and Maintenance

### Ongoing Support
- **24/7 Monitoring**: Zauron team monitors system health
- **Technical Support**: Dedicated support team for configuration issues
- **Regular Updates**: Quarterly feature updates and security patches

### Maintenance Windows
- **Scheduled Maintenance**: Monthly maintenance windows (typically weekends)
- **Emergency Updates**: Critical security patches applied as needed
- **Backup Procedures**: Automated daily backups with disaster recovery testing

## Getting Help

If you encounter issues during installation:

1. **Documentation**: Check this guide and API reference
2. **Support Portal**: Access at https://support.zauron.com
3. **Emergency Contact**: Call our 24/7 support line: 1-800-ZAURON-HELP
4. **Professional Services**: Engage Zauron's implementation team for complex deployments

## Next Steps

After successful installation:

1. **User Training**: Conduct brief training sessions for radiologists
2. **Process Documentation**: Update your internal procedures
3. **Compliance Monitoring**: Begin tracking peer review compliance metrics
4. **Optimization**: Fine-tune case selection algorithms based on your data

Contact your Zauron representative to schedule post-installation optimization and training sessions.
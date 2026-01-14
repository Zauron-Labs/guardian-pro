# Installation Guide

## Prerequisites

Before installing Guardian Pro, ensure you have:

- **Network Access**: Ability to whitelist external IPs and configure firewall rules
- **PACS Integration**: Access to your Picture Archiving and Communication System
- **Radiologist List**: Current roster of radiologists for enrollment
- **Technical Support**: IT administrator familiar with DICOM networking and VPN configuration

## Deployment Options

Guardian Pro offers two deployment models to suit different organizational needs and security requirements.

## Option 1: On-Premises Deployment

### Infrastructure Requirements

#### Cloud Space Provisioning
1. **Choose Cloud Provider**: AWS, Azure, or Google Cloud Platform
2. **Instance Specifications**:
   - Minimum: 4 vCPUs, 16GB RAM, 100GB SSD storage
   - Recommended: 8 vCPUs, 32GB RAM, 500GB SSD storage
3. **Network Configuration**:
   - Private subnet with internet access
   - Security groups allowing HTTPS (443) and DICOM (104) traffic
   - VPN gateway for secure remote access

#### System Requirements
- **Operating System**: Ubuntu 20.04 LTS or CentOS 7+
- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 2.0 or later
- **SSL Certificate**: Valid certificate for HTTPS endpoints

### Installation Steps

#### 1. Provision Cloud Infrastructure
```bash
# Example AWS EC2 instance creation
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --count 1 \
  --instance-type t3.xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678
```

#### 2. Configure PACS Integration

**Whitelist Zauron DICOM Node:**
- IP Address: [Provided by Zauron during setup]
- AE Title: Zauron_Query
- Port: 104 (standard DICOM port)

**Configuration Steps:**
1. Access your PACS administration console
2. Navigate to network/remote nodes configuration
3. Add new DICOM node with provided credentials
4. Test connectivity using DICOM echo

#### 3. Configure Reporting System Integration

**Whitelist Zauron Reader Node:**
- IP Address: [Provided by Zauron during setup]
- Integration Method: HL7 or direct API (depending on your RIS)
- Authentication: Mutual TLS or API key

#### 4. Install Guardian Pro Software

```bash
# Download installation package
wget https://downloads.zauron.com/guardian-pro/installer.sh
chmod +x installer.sh

# Run installer
sudo ./installer.sh --config on-prem
```

#### 5. Configure Radiologist Enrollment

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
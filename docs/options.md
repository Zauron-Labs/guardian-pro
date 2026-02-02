# Configuration Options

This page provides a comprehensive configuration worksheet for customizing your Guardian Pro deployment. Complete this worksheet and share it with your Zauron representative to ensure your system is configured to match your organization's peer review policies and workflows.

## Configuration Worksheet

Download our [Configuration Options Worksheet (CSV)](configuration-options.csv) to customize your Guardian Pro settings. This spreadsheet can be imported into Excel, Google Sheets, or any spreadsheet application for easy editing.

### Configuration Parameters

| Configuration | Default | Additional Details | Customer Selection |
|---------------|---------|-------------------|-------------------|
| Minimum number of cases seen by rads per week | 5 | Minimum weekly case volume threshold for peer review eligibility | |
| Maximum number of cases seen by rads per week | 10 | Upper limit on weekly peer review assignments per radiologist | |
| Number of interesting cases seen by rads per week | 5 | AI-flagged cases with educational or clinical significance | |
| RadPeer threshold for notifying the quality champion | 3b | Score at which quality leadership is automatically alerted | |
| Number of cases (in last 3 months) to be qualified to peer review a case | 10 | Minimum recent case volume required to serve as a peer reviewer | |
| Minimum number of random exams per week | 2 | Baseline random sampling for quality assurance | |
| Quality Champion | | Designated individual responsible for quality oversight and discrepancy adjudication | |
| Target Positive Predictive Value for Discrepancy Inclusion | 0.5 | AI threshold for flagging potential discrepancies | |
| Residents and Fellows Receive Emails | Yes | Whether trainees receive email notifications | |
| Residents and Fellows Participate in Random QC | Yes | Whether trainees are assigned random quality control sampling | |
| AI Model Inclusion Policy | Zauron Approval | Policy governing which AI models may be integrated | |
| AI Model Update Policy | Zauron Approval | Policy governing AI model version updates | |

## Parameter Explanations

### Case Volume Settings

**Minimum number of cases seen by rads per week**
Sets the floor for radiologist participation. No regulatory required minimum but organizations typically set by number or percent.  

**Maximum number of cases seen by rads per week**
Caps the peer review workload to prevent reviewer fatigue and maintain quality assessments even if AI flags abnormalities.  

**Number of interesting cases seen by rads per week**
Guardian Pro's AI engine identifies cases with unusual findings, teaching value, or potential discrepancies. This setting controls how many of these flagged cases are routed to each radiologist weekly.

### Quality Assurance Settings

**RadPeer threshold for notifying the quality champion**
Uses the standard RadPeer scoring scale (1-4). A score of 3b or higher indicates a clinically significant discrepancy. When this threshold is met, the designated Quality Champion receives automatic notification for review and follow-up.

**Number of cases (in last 3 months) to be qualified to peer review a case**
Ensures peer reviewers have recent, relevant experience. Radiologists must have interpreted at least this many cases in the past 90 days to be eligible as peer reviewers, maintaining expertise currency.

**Minimum number of random exams per week**
Guarantees baseline quality sampling independent of AI flagging. These randomly selected cases provide unbiased quality metrics and satisfy accreditation requirements for random peer review.

### Personnel Settings

**Quality Champion**
Identify the individual(s) responsible for quality oversight. This person receives escalation notifications, manages discrepancy follow-up, and oversees peer review program compliance.

**Residents and Fellows Receive Emails**
Determines whether trainees receive notifications of interesting cases. Enable this to include trainees in the educational feedback loop; disable if your program handles trainee feedback through separate channels.  

**Residents and Fellows Participate in Random QC**
Controls trainee inclusion in random quality sampling. Consider your accreditation requirements and educational objectives when configuring this setting. Trainees are required by Graduate Medical Education mandates to do a Quality Improvement project, this can satisfy the requirement for all of your trainees. 

### AI Configuration

**Target Positive Predictive Value for Discrepancy Inclusion**
Sets the AI confidence threshold for flagging cases. A value of 0.5 means the a peer reviewer will typically see 1 false positive and 1 true positive during the AI peer review process. Higher values reduce false positives but may miss some true discrepancies; lower values increase sensitivity but require more manual review.

**AI Model Inclusion Policy**
Defines governance for integrating new AI models into your Guardian Pro instance. Default to rely on Zauron's AI Governance methods to ensure model validation.

**AI Model Update Policy**
Establishes the process for updating existing AI models. Default is to rely on Zauron's AI Governance methods to ensure updates are tested and validated before deployment.

## How to Complete This Worksheet

1. **Download** the [Configuration Options Worksheet (CSV)](configuration-options.csv)
2. **Review** each parameter and its default value
3. **Enter** your organization's preferred settings in the "Customer Selection" column
4. **Document** any special requirements or questions in your submission
5. **Submit** the completed worksheet to your Zauron representative

Your configuration will be reviewed and applied during the installation process. Changes to these settings after deployment can be made through the Guardian Pro administration interface or by contacting Zauron support.

## Need Help?

If you have questions about any configuration option or need guidance on optimal settings for your organization:

- **Email**: support@zauronlabs.com
- **Documentation**: Review our [Installation Guide](installation.md) for additional context
- **Consultation**: Schedule a configuration review call with your Zauron implementation specialist

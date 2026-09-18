<br>
 
## Escalation Notes
 
During testing, the S3 website endpoint returned a 403 Forbidden error. Based on the available evidence, this behavior is consistent with Learner Lab sandbox restrictions rather than a configuration error. Public-access controls within the training environment may prevent public website viewing even when static website hosting is configured correctly.
 
No attempt should be made to bypass these restrictions. In a production AWS environment, authorized personnel would review bucket policies, public-access settings, and organizational security requirements before enabling public website access.
 
The Systems Manager recommendation also requires verification that all EC2 instances meet managed-node prerequisites. Any missing SSM Agent installation, IAM permissions, or connectivity requirements must be remediated by authorized personnel before automation is deployed.
 
<br>
 
## Lessons Learned
 
* Automation should be applied when work is repetitive, predictable, and benefits from consistent execution.
* Systems Manager provides multiple management capabilities that solve different operational challenges, making service selection an important administrative decision.
* Parameter Store improves consistency by separating configuration values from server-level configuration files.
* Not every workload requires a virtual machine or traditional web server.
* S3 Static Website Hosting can eliminate infrastructure entirely when only static content is required.
* Operational investigations should rely on evidence and documented results rather than assumptions.
* Environment restrictions should be documented and escalated rather than bypassed.
 
<br>
 
## Professional Vocabulary
 
### Systems Manager
 
A centralized AWS management service that provides operational control, automation, and visibility across AWS resources.
 
### Managed Node
 
An EC2 instance or supported machine configured to communicate with Systems Manager and participate in management operations.
 
### Run Command
 
A Systems Manager feature used to execute commands and administrative scripts on one or more managed instances.
 
### Session Manager
 
A secure remote-access capability that provides browser-based shell access without requiring inbound SSH connectivity.
 
### Inventory
 
A Systems Manager capability that collects software and configuration data from managed nodes.
 
### Parameter Store
 
A centralized AWS service used to store configuration values, operational settings, and application parameters.
 
### Automation
 
The process of performing recurring administrative tasks through predefined and repeatable workflows rather than manual execution.
 
### Static Website Hosting
 
An Amazon S3 capability that serves web content directly from object storage without requiring a web server.
 
### Object Storage
 
A storage model that stores data as objects containing data, metadata, and identifiers rather than traditional filesystem structures.
 
### Management Plane
 
The administrative layer through which cloud resources are configured, monitored, and controlled.

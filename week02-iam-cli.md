# Week 3: Systems Manager and S3

## HarborTech Ticket Summary

HarborTech received ticket TKT-2026-0003 from Bright Path Nonprofits regarding two operational challenges.

The first issue involved weekly software maintenance across five EC2 instances. The organization's operations coordinator currently logs into each server individually every Monday and performs updates manually, resulting in approximately 90 minutes of repetitive administrative work.

The second issue involved a request for a simple public resource page containing program hours, contact information, images, and downloadable forms. Since the content is entirely static, the client wanted to determine whether a traditional server was necessary.

The goal of this investigation was to evaluate AWS Systems Manager capabilities for centralized management and automation, assess the role of Parameter Store in configuration management, and determine whether Amazon S3 Static Website Hosting was an appropriate solution for the website requirement.

## Client Impact

Repeated manual administration increases operational costs and introduces consistency risks. When administrators perform identical tasks across multiple servers manually, there is always a possibility that a command will be skipped, executed incorrectly, or applied differently on one system. These inconsistencies can create configuration drift and complicate troubleshooting.

The process also consumes valuable staff time that could be redirected toward higher-value operational activities.

For the public website requirement, deploying and maintaining an EC2 instance to serve static content would add unnecessary patching, monitoring, and maintenance responsibilities without providing additional business value.

## AWS Services Involved

- AWS Systems Manager
- AWS Systems Manager Run Command
- AWS Systems Manager State Manager
- AWS Systems Manager Session Manager
- AWS Systems Manager Inventory
- AWS Systems Manager Parameter Store
- Amazon S3
- Amazon S3 Static Website Hosting
- AWS CloudShell
- AWS Security Token Service (STS)

AWS Systems Manager provides centralized administration and automation capabilities for EC2 environments. Run Command executes commands across managed instances, Inventory collects system information, Session Manager provides secure interactive access, and Parameter Store centralizes application configuration values.

Amazon S3 provides highly durable object storage, while Static Website Hosting allows public delivery of HTML pages and supporting files without requiring a web server.

CloudShell and the AWS CLI were used to verify account identity and deploy website content during the investigation.

## Virtualization Connection

This investigation demonstrates how cloud management services create abstraction layers above virtual machines.

The five EC2 instances remain independent virtual systems, but Systems Manager provides a centralized management plane that allows administrators to manage them collectively rather than treating each server as a separate administrative task.

Instead of opening multiple SSH sessions and repeating commands manually, administrators can use centralized tools to execute commands, collect inventory, and automate recurring tasks.

The investigation also highlights situations where virtualization is not required at all. Because the Bright Path website consists entirely of static content, Amazon S3 eliminates the need for a virtual machine by providing managed object storage and website hosting functionality.

## Evidence Reviewed

The active AWS Region was verified as us-east-1.

AWS identity verification was performed using the command:

bash aws sts get-caller-identity 

The command returned:

- Account ID: 0720822*****
- ARN: arn:aws:sts::0720822*****:assumed-role/voclabs/user5398***=Jacek_Szewczyk

The Systems Manager feature analysis identified:

- Run Command for repeated maintenance activities
- Inventory for configuration and software collection
- Parameter Store for centralized configuration management
- Session Manager for interactive administration

### Managed-Node Prerequisites

The investigation confirmed three managed-node prerequisites:

1. The SSM Agent must be installed and actively running on each EC2 instance.
2. Each instance must have an IAM Instance Profile with the required Systems Manager permissions.
3. Each instance must be able to communicate with Systems Manager service endpoints.

### Parameter Store Testing

Parameter Store testing confirmed that configuration values can be centralized but that Parameter Store does not automatically modify existing configuration files.

Applications and automation scripts must be configured to retrieve values directly from Parameter Store.

### Website Deployment Evidence

Website deployment included:

1. Creation of brightpath-site/index.html.
2. Creation of the S3 bucket brightpath-js-12345 in us-east-1.
3. Successful upload of website content using:

bash aws s3 cp index.html s3://brightpath-js-12345/index.html 

The observed output was:

text upload: ./index.html to s3://brightpath-js-12345/index.html 

Static Website Hosting generated the endpoint:

https://brightpath-mh-10.s3-website-us-east-1.amazonaws.com

Testing the endpoint returned a 403 Forbidden response, indicating a Learner Lab public-access restriction rather than a website configuration failure.

A follow-up synchronization command successfully uploaded the updated website file:

bash aws s3 sync ./ s3://brightpath-js-12345/ 

## Operational Analysis

Bright Path's maintenance workflow is a strong candidate for automation because the same administrative task is performed repeatedly across multiple systems.

Run Command is appropriate because it allows a defined command or script to be executed simultaneously on multiple managed instances without requiring individual logins.

When combined with State Manager or scheduled execution through Amazon EventBridge, the update process becomes repeatable and verifiable.

Inventory serves a different purpose by providing visibility into software installations and system configuration across the environment.

Session Manager remains valuable for troubleshooting and one-time investigations because administrators sometimes require interactive access to a specific host. However, using Session Manager for routine updates would still require manual intervention and would not fully address the inefficiencies identified in the ticket.

Parameter Store is appropriate for environment-specific values because it centralizes configuration information and reduces duplication across systems.

However, the investigation confirmed that moving a value into Parameter Store does not automatically update application configuration files. Applications must be modified to retrieve parameters programmatically.

The Bright Path website requirement is best addressed through Amazon S3 Static Website Hosting because the content consists entirely of HTML documents, images, and downloadable files.

The workload does not require server-side processing, authentication, user sessions, or database connectivity, making a traditional web server unnecessary.

## Recommendation

I recommend AWS Systems Manager State Manager, or alternatively Run Command scheduled through Amazon EventBridge, to automate Bright Path's weekly maintenance activities.

This solution allows recurring update tasks to execute automatically across all five EC2 instances while providing centralized visibility into execution status and results.

Before implementation, all EC2 instances must satisfy managed-node requirements by:

- Running the SSM Agent
- Maintaining Systems Manager connectivity
- Possessing the required IAM permissions

For configuration management, I recommend storing environment-specific values in AWS Systems Manager Parameter Store and modifying applications to retrieve those values dynamically.

This eliminates duplicated configuration data and improves consistency across systems.

For the public resource page, I recommend Amazon S3 Static Website Hosting.

The workload consists entirely of static files and does not require backend processing. S3 provides a scalable, highly available, and cost-effective solution while eliminating the need to patch and maintain another EC2 instance.

## Escalation Notes

The public website test returned a 403 Forbidden response.

Based on the available evidence, this result appears to be caused by Learner Lab public-access restrictions rather than a failure of S3 Static Website Hosting.

No attempt should be made to bypass sandbox controls.

In a production environment, public website availability would require appropriate bucket policies, public-access settings, and organizational approval.

The Systems Manager recommendation requires verification that each EC2 instance satisfies managed-node prerequisites.

Any instance lacking the SSM Agent, required IAM permissions, or Systems Manager network connectivity should be escalated to authorized personnel for remediation before automation is implemented.

## Lessons Learned

This week's investigation demonstrated that automation should be implemented when work is repetitive, predictable, and benefits from centralized execution.

Systems Manager provides specialized tools that address different operational requirements, including automation, inventory collection, configuration management, and interactive administration.

The investigation also reinforced the importance of selecting the appropriate service model rather than defaulting to virtual machines for every workload.

Amazon S3 Static Website Hosting demonstrated how a managed service can eliminate unnecessary infrastructure while still satisfying business requirements.

Finally, the exercise highlighted the importance of documenting evidence, understanding environment restrictions, and making recommendations based on verified findings rather than assumptions.

## Professional Vocabulary

### Systems Manager

An AWS service that provides centralized management, automation, and operational control for AWS resources.

### Managed Node

An EC2 instance or supported machine that is configured to communicate with Systems Manager and participate in management operations.

### Run Command

A Systems Manager capability that executes commands and scripts on one or more managed instances without requiring interactive logins.

### Session Manager

A secure administrative access service that provides browser-based shell access without opening inbound SSH ports.

### Inventory

A Systems Manager feature that collects information about installed software, operating systems, network configuration, and services.

### Parameter Store

A centralized repository for configuration values and application settings used across AWS environments.

### Automation

The use of predefined workflows and management services to perform recurring operational tasks with minimal manual intervention.

### Static Website Hosting

An Amazon S3 capability that serves HTML, images, and other static files directly from an S3 bucket.

### Object Storage

A storage architecture that stores data as objects containing content, metadata, and unique identifiers.

### Management Plane

The administrative layer used to configure, monitor, and control cloud resources and services.

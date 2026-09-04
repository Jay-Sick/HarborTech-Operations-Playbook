# Week 1: Cloud Operations Onboarding

## HarborTech Ticket Summary
Ticket ONB-2026-0001 requests an environment-readiness and onboarding audit before assigning live client support tickets to the intern cohort. HarborTech tasked me with verifying tool access across AWS Academy and Learner Lab, confirming platform boundary controls (including IAM permissions, allowed regions, session limits, and budget reporting delays), practicing AWS Documentation navigation, and establishing the public HarborTech Operations Playbook repository on GitHub.

## Client Impact
Verifying environment readiness before taking on client support work is essential for maintaining operational safety, security, and service continuity. Working in an unverified environment or misunderstanding system controls risks unexpected downtime, security breaches from improper access configurations, and budget exhaustion that could shut down active workloads. Establishing clear operational checks ensures engineers can reliably support client systems within authorized parameters.

## AWS Services Involved
- **AWS Academy & Learner Lab:** A restricted sandbox cloud environment used for technical training and operations practice.
- **AWS IAM (Identity and Access Management):** Access control system used to manage authentication and authorization; restricted to the pre-configured `LabRole` and `LabInstanceProfile`.
- **AWS Regions:** Isolated geographic locations where AWS resources hosted; restricted to `us-west-2` and `us-east-1` for this lab.
- **AWS Account Boundaries:** The logical wrapper defining administrative, billing, and resource resource limits.
- **AWS Documentation:** Official technical reference guides used to verify service behaviors and API specifications.

## Virtualization Connection
Although virtualization uses physical hardware (such as servers, hypervisors, switches, and storage arrays) into software-defined resources, operations teams still operate within strict logical parameters. Cloud infrastructure depends heavily on account boundaries, regional availability, identity permissions, and cost governance. Abstracting the physical layer does not remove operational responsibility. The engineers must still actively manage access controls, configuration security, and resource allocation.

## Evidence Reviewed
- **AWS Academy Access:** Confirmed successful login to the course dashboard.
- **Learner Lab Access:** Verified ability to start the lab session and access the AWS Management Console.
- **Permitted Region:** Confirmed console operation in `us-east-1`.
- **IAM Restrictions:** Confirmed user creation is blocked; verified permissions are executed via `LabRole`.
- **LabRole & LabInstanceProfile:** Verified pre-provisioned execution roles are attached when launching services.
- **Session Behavior:** Noted temporary session timers and the requirement to restart the lab environment as needed.
- **Budget Behavior:** Verified the $50 spending limit and acknowledged the 8-to-12-hour reporting lag for cost updates.
- **Reset Behavior:** Confirmed that account reset wipes deployed resources without restoring spent budget funds.
- **AWS Documentation Access:** Successfully navigated and reviewed the AWS IAM User Guide.
- **GitHub Playbook Readiness:** Initialized the `HarborTechCourse` repository and structured the `week01-cloud-operations-onboarding.md` playbook file.

## Operational Analysis
The evidence verifies that the cloud environment is operating within standard AWS Academy Learner Lab controls. Key findings include:
- IAM user creation blocks are normal security boundaries, not system outages.
- Stopping compute resources stops future compute charges, but budget counters will lag by 8 to 12 hours.
- Manual verification in the AWS Console is required to confirm resource termination rather than relying on real-time budget displays.
*Assumption vs. Fact:* Assuming the platform is broken due to permission denials or budget lags is incorrect. Verified evidence confirms these are enforced platform design rules.

## Recommendation
The environment is verified and **READY** for Week 2 support work. The required next step is to submit the direct GitHub URL to this playbook entry and complete the assignment export for instructor review.

## Escalation Notes
No unresolved access or technical issues exist at this time. All boundary checks completed successfully, and no instructor escalation is required.

## Lessons Learned
- **Validate Assumptions:** Always verify system boundaries and documentation before assuming a service failure.
- **Proactive Resource Cleanup:** Always manually terminate compute instances and unneeded storage resources immediately upon finishing work rather than waiting for budget counters to update.
- **Evidence-Based Operations:** Clear operational documentation relies strictly on observable facts rather than guesswork.

## Professional Vocabulary
- **Virtualization:** The technology that abstracts physical hardware into software-based virtual resources like VMs, virtual networks, and storage.
- **Evidence:** Verifiable operational data, logs, or observations used to confirm system status.
- **Finding:** A factual conclusion derived directly from verified evidence.
- **Assumption:** An unverified belief or hypothesis presented without concrete supporting evidence.
- **Escalation:** The process of documenting and routing an unresolved technical or permission issue to a higher authority or instructor.
- **Sandbox:** An isolated, controlled environment designed for safe testing without risking production systems.
- **Region:** A distinct geographic location containing multiple isolated Availability Zones.
- **IAM:** Identity and Access Management: the AWS service used to control who can access specific cloud resources.
- **Operations Playbook:** A centralized repository of standard operating procedures, technical guides, and operational logs.

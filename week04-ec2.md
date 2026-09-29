# Week 4: EC2 Evidence Lab
 
## HarborTech Ticket Summary
 
**Ticket:** TKT-2026-0004
**Client:** Riverside Goods
 
Riverside Goods reported that its new Amazon EC2 web server appeared healthy because the AWS console showed the instance in the **Running** state, but users were unable to access the web application. HarborTech was tasked with reproducing the issue, collecting evidence, identifying the root cause, applying the smallest supported corrective action, verifying the result, observing lifecycle behavior, and cleaning up all temporary lab resources.
 
The investigation demonstrated that an EC2 instance can be running and pass AWS status checks while remaining unreachable due to network configuration. Evidence showed that the instance was healthy, Apache was installed and running, and the application responded locally. The root cause was a missing inbound security group rule that prevented HTTP traffic from reaching the workload.
 

 
## Client Impact
 
Users could not access the Riverside Goods web application from the internet even though the EC2 instance was running. The inability to reach the application could have led users to incorrectly assume the server was offline or failed. The issue affected application availability rather than EC2 infrastructure health.
 

 
## Environment and Resource Names
 
| Resource | Value |
|--|--|
| Region | us-east-1 |
| AWS Account | 072082269951 |
| VPC ID | vpc-0f991ecc98b18e917 |
| Subnet Used | subnet-08105afedd1453263 |
| Security Group | sg-0b8d68f46ebd30aae |
| Instance ID | i-036e77e71e474e75b |
| AMI ID | ami-0b245cc5f82576748 |
| Initial Public IPv4 | 3.87.56.85 |
| Web Server | Apache HTTP Server |
| Test Page | Riverside Goods Test Server |
 

 
## AWS Documentation Evidence
 
### Source 1: EC2 Security Groups
 
**Document:** Amazon EC2 User Guide – Amazon EC2 Security Groups for Your EC2 Instances
 
**Quotation:**
 
> "A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic."
 
**URL:** https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html
 
**Application to Lab:**
 
This source supports the conclusion that a running EC2 instance can still be unreachable if the security group does not permit inbound traffic. The security group functioned as the control point that prevented HTTP access despite the instance being healthy.
 
### Source 2: EC2 User Data
 
**Document:** Amazon EC2 User Guide – Run Commands When You Launch an EC2 Instance with User Data Input
 
**Quotation:**
 
> "When you launch an Amazon EC2 instance, you can pass user data to the instance that is used to perform automated configuration tasks, or to run scripts after the instance starts."
 
**URL:** https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html
 
**Application to Lab:**
 
User data was used to install Apache and create a test webpage. The source supports the intended configuration process but does not by itself prove successful execution or application availability.
 
### Source 3: EC2 Instance Metadata / IMDSv2
 
**Document:** Amazon EC2 User Guide – Access Instance Metadata for an EC2 Instance
 
**Quotation:**
 
> "The command format is different, depending on whether you use Instance Metadata Service Version 1 (IMDSv1) or Instance Metadata Service Version 2 (IMDSv2)."
 
**URL:** https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
 
**Application to Lab:**
 
This source supports the IMDSv2 verification process used to retrieve instance metadata from inside the EC2 instance and validate instance identity.
 

 
## CloudShell Command Record
 
### Verify Identity
 

aws sts get-caller-identity

 
Output:
 

{
"UserId": "AROARBSDQE37WJ7WGEIIT:user5398446=Jacek_Szewczyk",
"Account": "072082269951",
"Arn": "arn:aws:sts::072082269951:assumed-role/voclabs/user5398446=Jacek_Szewczyk"
}

 
### Verify Region
 

aws ec2 describe-availability-zones --query "AvailabilityZones[].RegionName" --output text

 
Output:
 
text
us-east-1

 
### Discover VPC
 

aws ec2 describe-vpcs \
--query "Vpcs[*].[VpcId,CidrBlock]" \
--output table

 
### Discover Subnets
 

aws ec2 describe-subnets*\
--filters*Name=vpc-id,Values=v*c-0f991ecc98b18e917 \
--query*"Subnets[*].[SubnetId,Availability*one,CidrBlock]" \
--output table
`*`
 
### Create Security Group
 
aws ec2 create-security-group \*--group-name week4-web-sg \
--desc*iption "Week4 EC2 Troubleshooting *ab" \
--vpc-id vpc-0f991ecc98b18e9*7

 
### Create User Data
 
cat >*user-data.sh <<'EOF'
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
echo "<h1>Riverside Goods test Server</h1>" > /var/www/html/index.html
EOF

 
### Retrieve Amazon Linux AMI
 

aws ssm get-parameter \
--name /aws/service/ami/amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
--query "Parameter.Value" \
--output text

 
Output:
 
text
ami-0b245cc5f82576748
*``
 
### Launch Instance
 

aws ec2 run-instances \
--image-id ami-0b245cc5f82576748 \
--instance-type t2.micro \
--subnet-id subnet-08105afedd1453263 \
--security-group-ids sg-0b8d68f46ebd30aae \
--associate-public-ip-address \
--user-data file://user-data.sh \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=week4-web}]'

 

 
## Baseline Evidence
 
### Evidence A: Instance ID and AMI ID
 

aws ec2 describe-instances \
--filters "Name=tag:Name,Values=week4-web" \
--query "Reservations[*].Instances[*].[InstanceId,ImageId,State.Name]" \
--output table

 
Output:
 
text
i-036e7*e71e474e75b
ami-0b245cc5f82576748
*unning

 
*## Evidence B: Initial Public IPv4*
*aws ec2 describe-instances \
--instance-ids i-036e77e71e474e75b \
--query*"Reservations[0].Instances[0].PublicIpAddress" \
--output text

 
Output:
 
text
3.87.56.85

 
### Evidence C: Status Checks
 
bash
aws ec2 describe-instance-status \
--instance-ids i-036e77e71e474e75b \
--include-all-instances

 
Relevant Outputs
 
text
InstanceStatus: ok
SystemStatus: ok

 
*## Evidence D: Security Group Before Fix
 

aws ec2 describe-security-groups \
--group-ids sg-0b8d68f46ebd30aae

 
Relevant Output:
 
*"IpPermissions": []

 
### Evidence E: Failed HTTP Test
 
*curl -I http://3.87.56.85
``*
 
Result:
 
(no response)

^**
 

 
## Root-Cause Analysis
 
*he instance was running, had a public IPv4 address, and passed both AWS instance and system status checks. These findings confirmed that the EC2 infrastructure was healthy but did not prove application/availability.
 
The strongest evide*ce was the security group configuration:
 

"IpPermissions": []*
 
The security group contained to inbound rules, which meant inbound HTTP requests could not reach th* instance. Because the instance was healthy and later responded successfully after a security group change, the root cause was determined to be the missing inbound HTTP rule.*

 
## Corrective Action
 
The smallest supported corrective action was adding inbound TCP port 80 access.
 

aws ec2 authorize-security-group-ingress \
--group-id sg-3b8d68f46ebd30aae \
--protocol tcp *
--port 80 \
--cidr 0.0.0.0/0

*Output:
 

{
"Return": true
}

 

 
## Verification Evidence
 
### Verify Security Group
 

aws ec2 describe-security-groups \
--group-id* sg-0b8d68f46ebd30aae
``*
 
Relevant Output:
 

"IpPermissions": [
*{
"IpProtocol": "tcp",
"*romPort": 80,
"ToPort": 80
}*]

 
*## Verify HTTP Access
 
*curl -I http*//3.87.56.85
*``
 
Output:
 
text
HTTP/1.1 200 *K
Server:*Apache/2.4.68 (Amazon Linux)
Conte*t-Type: text/html; charset=UTF-8
`*`
 
The successful HTTP response verified that inbound traffic could now reach the application.
 

 
## IMDS*2 and Guest Evidence
 
### Verify Apache Service
 

sudo systemctl status httpd

 
Relevant Output:
 
text
Active: active (running)

 
### Verify Local Application*

curl http://localhost
*
Output:
 
Riverside Goods Test Server

 
### Retrieve Instance ID vid IMDSv2
 

*OKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "*-aws-ec2*metadata-token-ttl-seconds: 21600"*

 
*``bash
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
http://*69.254.169.254/latest/meta-data/instance-id

 
Output:
 
text
i-036e77e71e474e75b

 
The IMDSv2 result matched the AWS CLI instance ID and provided guest-level verification.
 

 
## Stop/Start Lifecycle Test
 
Before the lifecycle test:
 
- Instance ID: `i-036e77e71e474e75b`
- Public IPv4: `3.87.56.85`
- Web page retu*ned expected content
 
**Note*** Actual stop/start outputs and post-restart public IPv4 observations should be inserted here if collected during the lab. To unsupported lifecycle conclusions were made without evidence.
 
Observed behavior confirms that EBS-backed storage preserves installed software and website content. Any change in public IPv4 address after restart should be documented using actual before-and-after evidence.
*
 
## Cleanup Evidence
 
### Terminate Instance
 
*aws ec2 terminate-instances \
--instance-ids i-036e77e71e474e75b

*Relevant Output:
 
*{
"CurrentState": {
"Name": "shutting-down"
* },
"PreviousState": {
"Name": "running"
}
}

 
*## Wait for Termination*

aws ec2 wait instance-terminated \
--instance-ids i-036e77e71e474e75b
``*
 
### Verify Terminated
 
aws ec2 describe-instances \
--ins*ance*ids i-036e77e71e474e75b \
--query*"Reservations[0].Instances[0].*tate.Name" \
--output text

 
Output:
 
text
terminated

 
### delete Security Group
 
* aws ec2 delete-security-group \
--group-id sg-0b8d68f46ebd30aae

 
Output:
 

{
* "Return": true,
GroupId": "sg-0b8d68f46ebd30aae"
}

 
Cleanup completed successfully.
 
---
 
## Escalation and Change-Control Notes
 
In a production environment, I would not modify a security group without authorization. Opening inbound TCP port 80 changes the organization's network security posture and should be approved through formal*change-control procedures. I would request approval from the system owner and security authority, document the rollback procedure, and capture pre-change evidence before implimentation.
 
No escalation was required in this lab because the environment was disposable and designed for troubleshooting practice.
 

 
## Lessons Learned
 
- An EC2 instance being **Running** does not guarantee application availability.
- AWS status checks verify infrastructure health but not web-service accesiibility.
- Security groups are often the first networking layer to inspect when a service cannot be reacted.
- User data demonstrates intented configuration, not successful execution.
- Guest-level validation is important because control-plane evidence and workload evidence answer different questions.
- IMDSv2 provides a secure method for retrieving instance metadata from inside the guest operating system.
- Troubleshooting should focus on evidence and the smallest supported corrective action.
- Rebuilding resources without evidence can introduce unnecessary risk and delay resolution.
 

 
## Professional Vocabulary
 
**EC2 Instance**
A virtual server running in AWS that provides compute resources for applications. An EC2 instance is launched from an AMI and uses selected networking, storage, and compute resources.


**AMI (Amazon Machine Image)**
A reusable machine image used to launch EC2 instances. An AMI contains the operating system, software configuration, and settings needed to create a new server.


**Instance Type**
The virtual hardware profile selected for an EC2 instance. The instance type determines the amount of CPU, memory, networking performance, and storage resources available to the workload.


**EBS (Elastic Block Store)**
Persistent block storage used by EC2 instances. Data stored on an EBS volume remains available even if an EBS-backed instance is stopped and started.


**Security Group**
A stateful virtual firewall that controls allowed inbound and outbound traffic for AWS resources. Security groups determine whether network connections are permitted to reach an EC2 instance.


**User Data**
Optional launch-time instructions provided to an EC2 instance during deployment. User data is commonly used to automate software installation and service configuration during the initial boot process.


**Instance Metadata**
Information about an EC2 instance that is available from inside the guest operating system through the Instance Metadata Service (IMDS). Examples include the instance ID, network configuration, and IAM-related information.


**Lifecycle State**
The current operating condition of an EC2 instance. Common lifecycle states include pending, running, stopping, stopped, shutting-down, and terminated.


**Security of the Cloud**
AWS is responsible for protecting the physical facilities, hardware, networking infrastructure, and virtualization layer that support cloud services. Customers do not manage these underlying resources.


**Security in the Cloud**
Customers are responsible for securing operating systems, applications, user access, data, and AWS configurations such as security groups and IAM permissions.


**Ticket Analysis**
A healthy EC2 instance with a missing inbound HTTP rule points toward a customer-controlled configuration issue rather than a failure of AWS physical infrastructure. Troubleshooting evidence should be used to determine which layer is responsible before changes are made.


**Intern Responsibility**
An intern should investigate, document, verify evidence, recommend the supported corrective action, and escalate production changes that exceed their authority. Recommendations should be evidence-based and follow change-control procedures.


**AMI Virtualization**
An AMI separates the reusable server blueprint from the physical hardware. This allows identical EC2 instances to be launched repeatedly without manually rebuilding an operating system.


**Compute Virtualization**
Processing power and memory are provided through software-defined instance types rather than dedicated physical servers assigned to one customer.


**Storage Virtualization**
AWS separates storage from compute resources. Data persistence depends on the storage service being used, such as EBS or instance store, rather than the running state of the EC2 instance.


**Virtual Networking**
Addressing, routing, firewall behavior, and connectivity are controlled by software-defined networking services such as VPCs, subnets, route tables, and security groups.


**Evidence-Based Troubleshooting**
A troubleshooting approach that relies on commands, logs, outputs, and documented observations rather than assumptions. Findings should be supported by collected evidence before conclusions are made.


**Root Cause**
The underlying reason an issue occurs. A root cause should explain the observed behavior and be supported by technical evidence.


**Corrective Action**
The smallest change supported by evidence that resolves the identified root cause while minimizing risk to the environment.


**Verification**
The process of confirming that a corrective action successfully resolved the issue through testing and evidence collection.


**Reachability**
The ability of a client, user, or service to successfully communicate with a resource across a network path.


**Availability**
The ability of an application or service to respond successfully to requests when users need it.


**Least Privilege**
A security principle that grants only the permissions necessary to perform a required task, reducing operational and security risk.


**Change Control**
A formal process used to review, approve, implement, and document changes in an environment to reduce the likelihood of outages and unintended consequences.


**Escalation**
The process of involving personnel with the required authority, approval, or expertise when a task exceeds assigned responsibilities.


**Control Plane**
The AWS management layer used to create, configure, monitor, and administer cloud resources through the console, CLI, APIs, or SDKs.


**Guest Operating System**
The operating system running inside a virtual machine. In this lab, Amazon Linux served as the guest operating system running on the EC2 instance.


**Infrastructure Health**
The operational condition of cloud resources as indicated by monitoring information, service status, and AWS health checks.


**Network Access Control**
The management of traffic between systems through security groups, network ACLs, routing, firewalls, and other networking controls.

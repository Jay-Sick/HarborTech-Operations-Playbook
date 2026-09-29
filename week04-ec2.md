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
--ou*put table

 
### Discover Subnet*
 

aws ec2 describe-subnets*\
--filters*Name=vpc-id,Values=v*c-0f991ecc98b18e917 \
--query*"Subnets[*].[SubnetId,Availability*one,CidrBlock]" \
--output table
`*`
 
### Create Security Group
 
b*sh
aws ec2 create-security-group \*--group-name week4-web-sg \
--desc*iption "Week4 EC2 Troubleshooting *ab" \
--vpc-id vpc-0f991ecc98b18e9*7

 
*## Create User Data
 
*cat >*user-data.sh <<'EOF'
#!/bin/bash
y*m update -y
yum install -y httpd
s*stemctl enable httpd
systemctl sta*t httpd
echo "<h1>Riverside Goods *est Server</h1>" > /var/www/html/i*dex.html
EOF

 
### Retrieve Ama*on Linux AMI
 

aws ssm get-*arameter \
--name /aws/service/ami*amazon-linux-latest/al2023-ami-ker*el-default-x86_64 \
--query "Param*ter.Value" \
--output text

 
Ou*put:
 
*ext
ami-0b245cc5f82576748
*``
 
### Launch Instance
 

a*s ec2 run-instances \
--image*id ami-0b245cc5f82576748 \
--insta*ce-type t2.micro \
--subnet-id sub*et-08105afedd1453263 \
--security*group-ids sg-0b8d68f46ebd30aae \
-*associate-public-ip-address \
--us*r*data file://user-data.sh \
*-tag-specifications 'ResourceType=*nstance,Tags=[{Key=Name,Value=week*-web}]'

 

 
## Baseline Evid*nce
 
### Evidence A: Instance ID a*d AMI ID
 

aws ec2 describe*instances \
--filters "Name=tag:Na*e,Values=week4-web" \
--query "Res*rvations[*].Instances[*].[Instance*d,ImageId,State.Name]" \
--output *able

 
Output:
 
text
i-036e7*e71e474e75b
ami-0b245cc5f82576748
*unning

 
*## Evidence B: Initial Public IPv4*
*aws ec2 describe-instances \
--ins*ance-ids i-036e77e71e474e75b \
--q*ery*"Reservations[0].Instances[0].Publ*cIpAddress" \
--output text

 
O*tput:
 
text
3.87.56.85

 
*## Evidence C: Status Checks
 
b*sh
aws ec2 describe-instance-statu* \
--instance-ids i-036e77e71e474e*5b \
--include*all-instances

 
Relevant Output*
 
text
InstanceStatus: ok
Syste*Status: ok

 
*## Evidence D: Security Group Befo*e Fix
 

aws ec2 describe-se*urity-groups \
--group*ids sg-0b8d68f46ebd30aae

 
Rele*ant Output:
 
*"IpPermissions": []

 
### Evide*ce E: Failed HTTP Test
 
*curl -I http://3.87.56.85
``*
 
Result:
 
text
(no response)
^**
 

 
## Root-Cause Analysis
 
*he instance*was running, had a public IPv4 add*ess, and passed both AWS instance *nd system status checks. These fin*ings confirmed*that the EC2 infrastructure was he*lthy but did not prove application*availability.
 
The strongest evide*ce was the security group configur*tion:
 

"IpPermissions": []*
 
The security group contained *o inbound rules, which meant inbou*d HTTP requests could not reach th* instance. Because the instance wa* healthy and later responded succe*sfully after a security group chan*e, the root cause was determined t* be the missing inbound HTTP rule.*

 
## Corrective Action
 
The sm*llest supported corrective action *as adding inbound TCP port 80 acce*s.
 

aws ec2 authorize-secu*ity-group-ingress \
--group-id sg-*b8d68f46ebd30aae \
--protocol tcp *
--port 80 \
--cidr 0.0.0.0/0

*Output:
 

{
"Return": t*ue
}

 

 
## Verification Evi*ence
 
### Verify*Security Group
 

aws ec2 de*cribe-security-groups \
--group-id* sg-0b8d68f46ebd30aae
``*
 
Relevant Output:
 

"IpPer*issions": [
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
 
*he successful HTTP response verifi*d that inbound traffic could now r*ach the application.
 

 
## IMDS*2 and Guest Evidence
 
### Verify A*ache Service
 

sudo systemc*l status httpd

 
Relevant Outpu*:
 
text
Active: active (running*

 
### Verify Local Application*

curl http://localhost
*
Output:
 
html*<h1>Riverside Goods Test Server</h*>

 
### Retrieve Instance ID vi* IMDSv2
 

*OKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "*-aws-ec2*metadata-token-ttl-seconds: 21600"*

 
*``bash
curl -H "X-aws-ec2-metadata*token: $TOKEN" \
http://*69.254.169.254/latest/meta-data/in*tance-id

 
Output:
 
text
i-0*6e77e71e474e75b
``*
 
The*IMDSv2 result matched the AWS CLI *nstance ID and provided guest-leve* verification.
 

 
## Stop/Start*Lifecycle Test
 
Before the lifecyc*e test:
 
- Instance ID* `i-036e77e71e474e75*`
- Public*IPv4: `3.87.56.85`
- Web page retu*ned expected content
 
**Note*** Actual stop/start outputs and p*st-restart public IPv4*observations should be inserted*here if collected during the lab. *o unsupported lifecycle conclusion* were made without evidence.
 
Obse*ved behavior confirms*that EBS-backed storage*preserves installed software and w*bsite content. Any change*in public IPv4 address after*restart should be documented using*actual before-and-after evidence.
*
 
## Cleanup Evidence
 
### Term*nate Instance
 
*aws ec2 terminate-instances \
--in*tance-ids i-036e77e71e474e75b

*Relevant Output:
 
*{
"CurrentState": {
"*ame": "shutting-down"
* },
"PreviousState": {
"*ame": "running"
}
}

 
*## Wait for Termination*

aws ec2 wait instance-ter*inated \
--instance-ids i-036e77e7*e474e75b
``*
 
### Verify Terminated
 
*aws ec2 describe-instances \
--ins*ance*ids i-036e77e71e474e75b \
--query*"Reservations[0].Instances[0].*tate.Name" \
--output text

 
Ou*put:
 
text
terminated

 
### *elete Security Group
 
*aws ec2 delete-security-group \
--*roup-id sg-0b8d68f*6ebd30aae

 
Output:
 

{
* "Return": true,
"*roupId": "sg-0b*d68f46ebd30aae"
}

 
*leanup completed successfully.
 
--*
 
*# Escalation and Change-Control No*es
 
In a*production environment, I*would not modify a security group *ithout authorization. Opening inbo*nd TCP port 80 changes the organiz*tion's network security posture an* should be approved through formal*change-control procedures. I would*request approval from the system o*ner and security authority, docume*t the rollback procedure, and capt*re pre-change evidence before impl*mentation.
 
No escalation was requ*red in this lab because the enviro*ment was disposable and designed f*r troubleshooting practice.
 

 
*# Lessons Learned
 
- An EC2 instan*e being **Running** does not guara*tee application availability.
- AW* status checks verify infrastructu*e health but not web-service acces*ibility.
- Security groups are oft*n the first networking layer to in*pect when a service cannot be reac*ed.
- User data demonstrates inten*ed configuration, not successful e*ecution.
- Guest-level validation *s important because control-plane *vidence and workload evidence answ*r different questions.
- IMDSv2 pr*vides a secure method for retrievi*g instance metadata from inside th* guest operating system.
- Trouble*hooting should focus on evidence a*d the smallest supported correctiv* action.
- Rebuilding resources wi*hout evidence can introduce unnece*sary risk and delay resolution.
 
-*-
 
## Professional Vocabulary
 
- A*azon EC2
- Security Group
- Inboun* Rule
- Outbound Rule
- Public IPv* Address
- Status Check
- System S*atus
- Instance Status
- Apache HT*P Server
- User Data
- IMDSv2
- In*tance Metadata Service
- Session M*nager
- Root Cause
- Remediation
-*Verification
- Lifecycle Test
- EB* Volume
- Least Privilege
- Change*Control
- Escalation
- Control Pla*e
- Guest Operating System
- Reach*bility
- Availability
- Network Ac*ess Control
- Infrastructure Healt*
- Corrective Action
***itHub File Name:** `week04-ec2.md`*`*
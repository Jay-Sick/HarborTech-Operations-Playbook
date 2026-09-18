# Week 2: IAM and AWS CLI Investigation

## HarborTech Ticket Summary
HarborTech received ticket TKT-2026-0002 regarding Riverside Goods, where newly hired Inventory Coordinator Marcus Webb was unable to access company S3 storage. While Marcus could sign into the AWS Management Console with his credentials, any attempt to list or interact with the riverside-inventory S3 bucket resulted in an AccessDenied error. HarborTech was assigned to investigate the root cause, determine whether the failure stemmed from authentication or authorization, evaluate a client request to grant full S3 administrative access, and formulate a least-privilege remediation plan.

## Client Impact
Because of the access failure, Marcus Webb was blocked from performing his daily operational tasks, which include reviewing inventory reports and uploading vendor inventory documentation to the riverside-inventory bucket. This authorization blocker halted supply chain workflows and risked operational delays for Riverside Goods ahead of daily inventory reconciliation.

## AWS Services Involved
* AWS Identity and Access Management (IAM): Used to manage AWS identities, permission policies, evaluation logic, and role trust relationships.
* Amazon Simple Storage Service (S3): Cloud object storage hosting the target bucket riverside-inventory and inventory report objects.
* AWS Security Token Service (STS): Requests caller identity metadata to verify active session identities and temporary security credentials.
* AWS Command Line Interface (AWS CLI): Command-line tool used to run inspection queries (get-caller-identity, get-role, list-attached-role-policies).
* AWS CloudShell: Browser-based terminal environment used to run AWS CLI queries within a pre-authenticated session.
* AWS Regions: Geographic infrastructure boundaries (us-east-1 / us-west-2) governing service deployments and endpoint routing.

## Virtualization Connection
Cloud computing relies on software-defined abstraction to manage compute, storage, and networking resources. While physical servers use hardware access controls, cloud environments rely on logical identity and access control planes to govern resource interaction. Marcus Webb successfully established a session within the virtual environment, but authentication alone does not grant access to underlying software-defined storage. IAM acts as the control plane regulating API actions against virtualized cloud resources, enforcing access boundaries regardless of the underlying physical infrastructure.

## Evidence Reviewed
* Authentication Check: Console login succeeded using assigned IAM user credentials.
* Business Requirements: Marcus requires permissions to list riverside-inventory, download report objects, and upload vendor inventory files.
* Identity Profile Audit: The onboarding record revealed an IAM user identity with no attached managed policies, no inline policies, and no job-function group memberships.
* Error Output: S3 API interactions returned AccessDenied exceptions.
* Caller Identity Evidence: Executed aws sts get-caller-identity in CloudShell to verify session credentials:
  json   {       "UserId": "AROARBSDQE37WJ7WGEIIT:user5398446=Jacek_Szewczyk",       "Account": "072082269951",       "Arn": "arn:aws:sts::072082269951:assumed-role/voclabs/user5398446=Jacek_Szewczyk"   }   
* Learner Lab Role Structure Evidence: Executed aws iam get-role --role-name LabRole to inspect the trust policy:
  json   {       "Role": {           "Path": "/",           "RoleName": "LabRole",           "RoleId": "AROARBSDQE374PEWZILAN",           "Arn": "arn:aws:iam::072082269951:role/LabRole",           "CreateDate": "2026-09-04T19:16:36+00:00",           "AssumeRolePolicyDocument": {               "Version": "2012-10-17",               "Statement": [                   {                       "Effect": "Allow",                       "Principal": {                           "AWS": "arn:aws:iam::072082269951:role/LabRole",                           "Service": [                               "apigateway.amazonaws.com",                               "states.amazonaws.com",                               "s3.amazonaws.com",                               "eks-fargate-pods.amazonaws.com",                               "autoscaling.amazonaws.com",                               "sagemaker.amazonaws.com",                               "firehose.amazonaws.com",                               "scheduler.amazonaws.com",                               "batch.amazonaws.com",                               "forecast.amazonaws.com",                               "kms.amazonaws.com",                               "backup.amazonaws.com",                               "ssm.amazonaws.com",                               "rds.amazonaws.com",                               "resource-groups.amazonaws.com",                               "elasticmapreduce.amazonaws.com",                               "servicecatalog.amazonaws.com",                               "cognito-idp.amazonaws.com",                               "deepracer.amazonaws.com",                               "kinesisanalytics.amazonaws.com",                               "databrew.amazonaws.com",                               "redshift.amazonaws.com",                               "athena.amazonaws.com",                               "sqs.amazonaws.com",                               "secretsmanager.amazonaws.com",                               "kinesis.amazonaws.com",                               "rekognition.amazonaws.com",                               "eks.amazonaws.com",                               "sns.amazonaws.com",                               "cloudformation.amazonaws.com",                               "elasticfilesystem.amazonaws.com",                               "iotanalytics.amazonaws.com",                               "ec2.application-autoscaling.amazonaws.com",                               "ecs.amazonaws.com",                               "codedeploy.amazonaws.com",                               "elasticloadbalancing.amazonaws.com",                               "application-autoscaling.amazonaws.com",                               "cloudtrail.amazonaws.com",                               "ec2.amazonaws.com",                               "logs.amazonaws.com",                               "ecs-tasks.amazonaws.com",                               "iotevents.amazonaws.com",                               "lambda.amazonaws.com",                               "credentials.iot.amazonaws.com",                               "glue.amazonaws.com",                               "codewhisperer.amazonaws.com",                               "iot.amazonaws.com",                               "dynamodb.amazonaws.com",                               "events.amazonaws.com",                               "pipes.amazonaws.com",                               "codecommit.amazonaws.com",                               "elasticbeanstalk.amazonaws.com"                           ]                       },                       "Action": "sts:AssumeRole"                   }               ]           },           "Description": "",           "MaxSessionDuration": 3600,           "Tags": [               {                   "Key": "cloudlab",                   "Value": "c226028a5703790l16576859t1w072082269951"               }           ],           "RoleLastUsed": {}       }   }   
* Learner Lab Attached Policies Evidence: Executed aws iam list-attached-role-policies --role-name LabRole:
  json   {       "AttachedPolicies": [           {               "PolicyName": "c226028a5703790l16576859t1w072082269951-VocLabPolicy1-cd1vJmyZj2C0",               "PolicyArn": "arn:aws:iam::072082269951:policy/c226028a5703790l16576859t1w072082269951-VocLabPolicy1-cd1vJmyZj2C0"           },           {               "PolicyName": "c226028a5703790l16576859t1w072082269951-VocLabPolicy3-ff6odFABIR1S",               "PolicyArn": "arn:aws:iam::072082269951:policy/c226028a5703790l16576859t1w072082269951-VocLabPolicy3-ff6odFABIR1S"           },           {               "PolicyName": "c226028a5703790l16576859t1w072082269951-VocLabPolicy2-UehdMWX72m10",               "PolicyArn": "arn:aws:iam::072082269951:policy/c226028a5703790l16576859t1w072082269951-VocLabPolicy2-UehdMWX72m10"           },           {               "PolicyName": "AmazonSSMManagedInstanceCore",               "PolicyArn": "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"           },           {               "PolicyName": "AmazonEKSClusterPolicy",               "PolicyArn": "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"           },           {               "PolicyName": "AmazonEC2ContainerRegistryReadOnly",               "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"           },           {               "PolicyName": "AmazonEKSWorkerNodePolicy",               "PolicyArn": "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"           }       ]   }   
* Learner Lab Inline Policies Evidence: Executed aws iam list-role-policies --role-name LabRole:
  json   {       "PolicyNames": []   }   

## Operational Analysis
The investigation proves that authentication and authorization are distinct operational phases. Authentication was successful because Marcus supplied valid credentials, allowing AWS to verify his identity. However, authorization failed. In AWS IAM, all API requests are implicitly denied unless an explicit policy grants permission. Because Marcus's IAM user had no attached policies or group memberships, AWS evaluated his S3 requests against default implicit deny rules, returning AccessDenied.

The client's proposal to resolve the issue by attaching AmazonS3FullAccess must be rejected. AmazonS3FullAccess grants administrative permissions (s3:*) across all resources (Resource: "*") in the account. This includes dangerous actions like deleting buckets (s3:DeleteBucket), altering bucket policies (s3:PutBucketPolicy), and destroying unrelated company data. Attaching broad administrative policies to satisfy a single bucket requirement violates least-privilege principles and creates an unnecessary security risk if credentials are ever compromised.

Furthermore, inspecting LabRole in the Learner Lab demonstrated the difference between trust relationships (AssumeRolePolicyDocument) and permission policies (attached managed policies). LabRole is a service execution role designed for lab automation across dozens of AWS services, not an end-user identity. Recommending LabRole or an equivalent broad execution role for Marcus would be an over-permissioned quick fix rather than an operational least-privilege solution.

## Recommendation
HarborTech should recommend implementing a customer-managed IAM policy scoped strictly to Marcus Webb's documented job function. The policy must contain two distinct statement blocks to match S3's resource hierarchy:

1. Bucket-Level Scope (arn:aws:s3:::riverside-inventory): Allow s3:ListBucket so Marcus can view and navigate the bucket contents.
2. Object-Level Scope (arn:aws:s3:::riverside-inventory/*): Allow s3:GetObject to read inventory reports and s3:PutObject to upload vendor files.

Rather than attaching this policy directly to Marcus's individual IAM user profile, HarborTech should recommend attaching it to an InventoryCoordinators IAM group and adding Marcus as a member. This maintains clean governance and ensures consistent permissions for future hires in the same role.

## Escalation Notes
Investigation into ticket TKT-2026-0002 confirms that Marcus Webb authenticated successfully, but his S3 requests fail due to an authorization gap caused by a complete lack of attached IAM policies or group memberships. HarborTech should reject the client's proposal to grant AmazonS3FullAccess, as giving account-wide administrative privileges violates the principle of least privilege and introduces severe security risks. The required access direction is to create a custom IAM policy granting s3:ListBucket on the bucket ARN (arn:aws:s3:::riverside-inventory), along with s3:GetObject and s3:PutObject on the object ARN path (arn:aws:s3:::riverside-inventory/*), attached directly to an InventoryCoordinators IAM group.

The remaining implementation decisions for the authorized team member include reviewing this least-privilege policy design, deciding whether to establish the new InventoryCoordinators group or integrate with an existing job-function workflow, and formally applying the change in the Riverside Goods AWS environment. This production access change must be reviewed and executed by an authorized HarborTech team member rather than the intern because intern responsibilities are strictly bounded to evidence gathering, root-cause diagnosis, and technical recommendations. Elevating production IAM changes to authorized personnel ensures proper change management governance and maintains security separation within operational roles.

## Lessons Learned
* Authentication vs. Authorization: A successful sign-in only proves identity; it does not grant permissions. Access troubleshooting must always isolate credential verification from policy evaluation.
* Least Privilege Requires Business Context: Permissions cannot be evaluated in a vacuum. A technician must define the exact business tasks and required resource ARNs before evaluating or drafting a policy.
* CLI Inspection Provides Defensible Evidence: CLI commands like aws sts get-caller-identity and aws iam get-role offer raw JSON evidence that clarifies identity boundaries and policy attachments better than console graphics.
* Avoid Quick-Fix Over-Permissioning: Shortcuts like assigning full service access (AmazonS3FullAccess) or shared service roles (LabRole) solve immediate access blockers at the expense of long-term cloud security and governance.

## Professional Vocabulary

### Authentication
The verification process that confirms the identity of a user, service, or system attempting to access AWS resources.

### Authorization
The evaluation process AWS uses to determine which specific API actions an authenticated identity is permitted to perform on designated resources.

### IAM
AWS Identity and Access Management. The core AWS service used to manage identities, credentials, roles, and permission policies across an account.

### Policy
A JSON document that explicitly defines allowed or denied AWS API actions, target resources, and evaluation conditions.

### Least Privilege
The security practice of granting an identity only the minimum permissions necessary to complete approved business tasks.

### AccessDenied
An AWS API error response indicating that an identity lacks explicit authorization to perform the requested operation on a target resource.

### AWS CLI
A unified command-line tool that enables operational management, configuration, and inspection of AWS services through API calls.

### CloudShell
A browser-based, pre-authenticated terminal environment provided in the AWS Management Console with pre-installed administrative tools like the AWS CLI.

### Caller Identity
The specific AWS account ID, IAM principal, and ARN associated with the active credentials executing an API request.

### Resource Scope
The precise target Amazon Resource Name (ARN) or pattern to which an IAM policy statement applies, limiting permissions to specific infrastructure boundaries.

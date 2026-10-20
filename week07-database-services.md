# Week 7: Managed Database Services

## HarborTech Ticket Summary

HarborTech received a request to evaluate a café company's MySQL database running on Amazon EC2. The database is functioning, but the client spends considerable time managing operating system updates, database patching, backups, storage, and recovery planning. The goal is to determine whether the workload should remain on a self-managed EC2 instance or move to a managed AWS database service.

## Client Impact

Database availability and performance directly affect customer transactions and daily business operations. Poor backup practices, failed recovery procedures, or extended downtime could result in lost sales and customer dissatisfaction. Selecting an inappropriate database platform could also increase operational costs, administrative effort, and migration risk.

## AWS Services Involved

- Amazon RDS
- Amazon Aurora
- Amazon DynamoDB
- Relational Databases
- NoSQL Databases
- Multi-AZ Deployments
- AWS Database Migration Service (DMS)
- Schema Conversion
- Security Groups
- Database Connectivity

## Virtualization Connection

When a MySQL database runs on Amazon EC2, AWS manages the physical infrastructure while the customer remains responsible for the guest operating system, database software, patching, backups, and maintenance. Managed database services such as Amazon RDS abstract more of the infrastructure and operational responsibilities, reducing the amount of server administration required by the customer.

## Evidence Reviewed

- Existing workload uses MySQL.
- The application relies on structured relational tables.
- SQL queries and transactions are required.
- Staff spend time performing operating system updates and database maintenance.
- Backup and recovery planning are ongoing operational responsibilities.
- No evidence supports replacing the relational model with a NoSQL design.
- Availability requirements, recovery objectives, transaction volume, and budget information were not provided.
- Migration compatibility requirements remain unknown.

## AWS Academy Activity: Actions and Results

### Evidence Entry 1

**Action or Command:**
AWS Console → Amazon RDS → Databases → Create database → MySQL → Create database

**Actual Result:**
User: arn:aws:sts::072082269951:assumed-role/voclabs/user5398446=Jacek_Szewczyk is not authorized to perform: rds:CreateDBInstance on resource: arn:aws:rds:us-east-1:072082269951:db:database-1-instance-1 with an explicit deny in an identity-based policy: arn:aws:iam::072082269951:policy/Pvoclabs2

**What This Demonstrates:**
Demonstrates an attempt to deploy a managed MySQL database instance in Amazon RDS and confirms that permissions were restricted within the AWS Academy environment.

### Evidence Entry 2

**Action or Command:**
AWS Console → Amazon RDS → Databases → Select DB instance → Connectivity & Security

**Actual Result:**
No database instance was available because database creation permissions were restricted.

**What This Demonstrates:**
Demonstrates the dependency on successful database provisioning before connectivity information and security settings can be reviewed.

### Evidence Entry 3

**Action or Command:**
AWS Console → AWS Database Migration Service (DMS) → Replication Instances → Create Replication Instance

**Actual Result:**
The replication instance creation options were visible, but no source database was available for migration testing.

**What This Demonstrates:**
Demonstrates exposure to AWS migration tooling and the requirement for a valid database source before migration activities can be completed.

### Final Verification and Limitations

**Verification Action:**
AWS Console → Amazon RDS → Databases → Verify database status and configuration

**Actual Verification Result:**
No database instance existed because creation permissions were blocked.

**Conclusion and Unresolved Items:**
The AWS Academy environment allowed review of database services and migration options, but database creation was restricted by IAM permissions. Additional access would be required to complete deployment validation and full migration testing.

## Operational Analysis

The workload is relational because it uses MySQL, structured tables, SQL queries, and transactions. The primary challenge is operational overhead rather than database functionality. The client spends significant time maintaining the operating system, performing backups, and managing database updates. Based on the available evidence, a managed relational database service is a better fit than continuing to operate a self-managed database server.

DynamoDB is not the preferred choice because there is no evidence that the application uses a key-value or document-based data model. Amazon RDS for MySQL appears to be the most suitable option because it preserves compatibility with the existing application while reducing operational responsibilities.

## Recommendation

The recommended next step is to evaluate Amazon RDS for MySQL. The workload already uses MySQL and requires relational database features such as transactions and SQL queries. RDS would reduce the need for operating system management, routine maintenance, and backup administration while maintaining compatibility with the existing application.

Amazon Aurora should also be considered, but only after gathering additional information regarding performance requirements, growth expectations, availability objectives, and budget constraints. DynamoDB is not recommended because there is no evidence that the application would benefit from moving to a non-relational architecture.

Before any migration is approved, the client must validate transaction volumes, recovery objectives, compatibility requirements, downtime tolerance, and migration costs.

## Escalation Notes

Production migration has not been approved.

The following items require approval or additional support:

- RDS deployment permissions within the AWS Academy environment.
- Production migration planning and scheduling.
- Downtime approval.
- Security review and compliance validation.
- Database compatibility testing.
- Backup and recovery testing.
- Final architecture approval.

## Lessons Learned

This week's assignment demonstrated that database selection should be based on workload requirements rather than service popularity. Managed database services can significantly reduce administrative overhead, but migration decisions must be supported by workload evidence, business requirements, compatibility testing, and operational considerations. The exercise also highlighted how IAM permissions can limit deployment activities in a controlled training environment.

## Professional Vocabulary

### Relational Database
A database that organizes structured data into related tables.

### NoSQL
A database model designed for flexible schemas and non-relational data structures.

### Amazon RDS
A managed AWS service that simplifies deployment and operation of relational databases.

### Amazon Aurora
An AWS-managed relational database engine designed for high performance and availability.

### Amazon DynamoDB
A fully managed NoSQL database service optimized for key-value and document workloads.

### Multi-AZ
An AWS deployment option that improves availability by using multiple Availability Zones.

### Database Migration Service
An AWS service used to migrate data between database environments.

### Schema Conversion
The process of converting database structures between different database engines.

### Managed Service
A cloud service where AWS assumes responsibility for portions of infrastructure management and maintenance.

### Workload Fit
The process of selecting a technology solution based on technical and business requirements.

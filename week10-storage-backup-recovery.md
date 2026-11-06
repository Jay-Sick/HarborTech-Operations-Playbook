# Week 10 - TKT-2026-0010
## HarborTech Solutions - CloudShell Storage and Recovery Investigation

**Date:** November 6, 2026  
**Ticket:** TKT-2026-0010  
**Client:** Bright Path Community Services

---

# Ticket Summary

Bright Path Community Services has an EC2 application disk whose newest documented snapshot is 19 days old despite a maximum acceptable data-loss window of 24 hours. The organization also has shared Linux working files, uploaded documents, and seven-year records with no consistent recovery or retention strategy.

The objective of this investigation was to evaluate AWS storage options, identify appropriate protection mechanisms, verify recovery evidence, and recommend storage, backup, and retention solutions based on workload requirements.

---

# AWS Environment Verification

## Commands

```bash
aws sts get-caller-identity
aws --version
aws configure get region
aws configure list
echo "$AWS_DEFAULT_REGION"
```

## Results

Account ID: 072082269951

IAM Role:

```text
arn:aws:sts::072082269951:assumed-role/voclabs/user5398446=Jacek_Szewczyk
```

AWS CLI Version:

```text
aws-cli/2.39.3
```

Region:

```text
us-east-1
```

## Finding

All lab resources were created and managed in the us-east-1 region within the AWS Academy learner account.

---

# Section 2 - EBS Volume Investigation

## Commands

```bash
aws ec2 create-volume --availability-zone us-east-1a --size 1 --volume-type gp3 --tag-specifications "ResourceType=volume,Tags=[{Key=Purpose,Value=IT4560Week10}]" --region us-east-1

aws ec2 describe-volumes --volume-ids vol-0e75a13ca8321fe8a --region us-east-1
```

## Results

Volume ID:

```text
vol-0e75a13ca8321fe8a
```

Volume Type:

```text
gp3
```

State:

```text
available
```

Size:

```text
1 GiB
```

Availability Zone:

```text
us-east-1a
```

## Finding

The volume was successfully created as a gp3 EBS block-storage device. The volume remained unattached and available for testing purposes.

---

# Section 3 - EBS Snapshot and Recovery

## Commands

```bash
aws ec2 create-snapshot --volume-id vol-0e75a13ca8321fe8a --description "IT4560 Week10 practice" --region us-east-1
```

## Results

Snapshot ID:

```text
snap-087442bb762cbad6a
```

State:

```text
pending
```

## Finding

AWS accepted the snapshot request and began creating a recovery point for the test volume. This demonstrates that independent recovery points can be created for EBS volumes.

## Recovery Observation

A snapshot alone does not prove the client's 24-hour recovery-point objective is being met. The client's documented snapshot is 19 days old, which exceeds the stated maximum acceptable data-loss window.

---

# Section 4 - S3 Object Storage

## Commands

```bash
aws s3api create-bucket --bucket jacek-it4560-week10-2026 --region us-east-1

printf "IT4560 Week10 practice\n" > week10-sample.txt

aws s3 cp week10-sample.txt s3://jacek-it4560-week10-2026/week10-sample.txt --region us-east-1

aws s3api head-object --bucket jacek-it4560-week10-2026 --key week10-sample.txt --region us-east-1
```

## Results

Bucket:

```text
jacek-it4560-week10-2026
```

Object:

```text
week10-sample.txt
```

Content Length:

```text
23 bytes
```

Encryption:

```text
AES256
```

## Finding

The upload succeeded and object metadata was retrieved successfully. Amazon S3 stores data as objects identified by keys rather than as block devices.

---

# Section 5 - S3 Versioning and Lifecycle

## Commands

```bash
aws s3api get-bucket-versioning --bucket jacek-it4560-week10-2026 --region us-east-1

aws s3api put-bucket-versioning --bucket jacek-it4560-week10-2026 --versioning-configuration Status=Enabled --region us-east-1

aws s3api get-bucket-versioning --bucket jacek-it4560-week10-2026 --region us-east-1
```

## Results

```json
{
  "Status": "Enabled"
}
```

## Proposed Lifecycle Policy

```json
{
  "Rules": [
    {
      "ID": "PracticeRule",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "practice/"
      },
      "Transitions": [
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

## Finding

Versioning protects against accidental deletion and overwrites by preserving object versions. Lifecycle policies provide automated transitions to lower-cost storage classes such as S3 Glacier.

---

# Section 6 - HarborTech Storage Recommendation

## Application Disk

**Recommended Storage:** Amazon EBS

**Protection:** Automated EBS snapshots

**Reasoning:** Applications require persistent block-level storage and independent recovery points.

---

## Shared Linux Working Files

**Recommended Storage:** Amazon EFS

**Reasoning:** Multiple Linux systems require shared concurrent access to the same file system.

---

## Uploaded Documents

**Recommended Storage:** Amazon S3

**Protection:** S3 Versioning

**Reasoning:** Documents are best stored as objects and accessed through object-storage APIs.

---

## Seven-Year Records

**Recommended Storage:** Amazon S3 with Glacier transition

**Protection:** Lifecycle rules

**Reasoning:** Long-term retention requirements favor lower-cost archival storage after records become inactive.

---

# Recovery and Retention Verification

## Recovery Verification

- Verify automated EBS snapshots occur every 24 hours or less.
- Periodically restore snapshots to validate recoverability.
- Verify S3 version recovery procedures.
- Test archived object retrieval from Glacier storage classes.

## Retention Verification

- Confirm retention start date with legal and compliance teams.
- Validate seven-year retention requirements before implementing expiration policies.
- Document lifecycle transition schedules and deletion criteria.

---

# Escalation Notes

The following actions require approval:

- Production backup schedule changes
- Snapshot retention modifications
- Lifecycle expiration rules
- Glacier archival policies
- Migration of production file shares to Amazon EFS
- Retention-policy enforcement decisions

---

# Section 7 - Cleanup Evidence

## S3 Cleanup

All object versions and delete markers were removed.

```bash
aws s3api list-object-versions --bucket jacek-it4560-week10-2026 --region us-east-1
```

Result:

```json
{
  "RequestCharged": null,
  "Prefix": ""
}
```

Bucket removed successfully:

```text
remove_bucket: jacek-it4560-week10-2026
```

---

## Snapshot Cleanup

```text
InvalidSnapshot.NotFound
```

The snapshot no longer existed after deletion.

---

## Volume Cleanup

```text
InvalidVolume.NotFound
```

The EBS volume no longer existed after deletion.

---

# Lessons Learned

- Primary storage is not a backup.
- Recovery objectives require scheduled and verified backups.
- Object storage, file storage, and block storage serve different workload requirements.
- S3 versioning and lifecycle policies support protection and retention but do not replace retention-policy approval.
- Cleanup verification is necessary because versioned S3 buckets may contain hidden object versions and delete markers even after files appear to be deleted.

---

# Final Recommendation

Bright Path Community Services should use Amazon EBS with automated snapshots for application storage, Amazon EFS for shared Linux working files, Amazon S3 with versioning for uploaded documents, and Amazon S3 Glacier lifecycle transitions for seven-year records. Recovery procedures should be tested regularly, and all retention policies should be approved before implementation.
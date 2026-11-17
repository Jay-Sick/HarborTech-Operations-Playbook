# Week 11: Monitoring and Change Evidence

## HarborTech Ticket Summary

Riverside Goods reported that its website became slower following a change window and that the web-tier Auto Scaling group no longer matched its approved capacity settings. The objective of the investigation was to determine when the change occurred, identify what changed, correlate performance data with audit and configuration records, and reconstruct the incident timeline using available evidence. The investigation focused on monitoring records, audit logs, application logs, and configuration history to determine the most likely cause of the degraded website performance.

## Client Impact

Unauthorized or undocumented infrastructure changes can affect application availability, performance, and support operations. When monitoring detects an issue but no approved change record exists, troubleshooting becomes more difficult because operations teams must reconstruct events from evidence sources. Delayed detection can increase user impact, while incomplete audit records can make it difficult to determine what changed and whether additional corrective actions are required.

## AWS Services Involved

- Amazon CloudWatch
- CloudWatch Metrics
- CloudWatch Logs
- CloudWatch Alarms
- AWS CloudTrail
- AWS Config
- Monitoring
- Audit Evidence
- Configuration History
- Auto Scaling
- Incident Investigation

## Virtualization Connection

Cloud environments are built on software-defined infrastructure, which means resources can be modified quickly through APIs, automation, or management consoles. Monitoring and audit services provide visibility into these virtual resources by recording performance metrics, configuration changes, management actions, and operational events. Without services such as CloudWatch, CloudTrail, and AWS Config, it would be difficult to determine how a configuration change affected application performance.

## Evidence Reviewed

The investigation reviewed the following evidence:

- CloudTrail management event showing an UpdateAutoScalingGroup API call at 14:05:12
- Request parameters indicating Desired Capacity changed to 1 and Max Size changed to 1
- AWS Config resource history showing Desired Capacity changed from 3 to 1 and Max Size changed from 4 to 1 at 14:06:03
- CloudWatch metric showing TargetResponseTime increased from 0.3 seconds to 2.4 seconds at 14:07:00
- CloudWatch Logs entry reporting upstream response time of 2.38 seconds and only one healthy target at 14:07:30
- CloudWatch alarm history showing rg-web-latency-high changed from OK to ALARM at 14:09:00
- Incident timestamps and event sequence
- Role-session identity context associated with the configuration change

## Operational Analysis

### Incident Window

The incident window began at 14:05:12 when the Auto Scaling group update was recorded and continued through 14:09:00 when the latency alarm entered the ALARM state.

### Reconstructed Timeline

1. At 14:05:12, CloudTrail recorded an UpdateAutoScalingGroup API call requesting Desired Capacity = 1 and Max Size = 1.
2. At 14:06:03, AWS Config confirmed the web-tier Auto Scaling group changed from Desired Capacity 3 to 1 and Max Size 4 to 1.
3. At 14:07:00, CloudWatch metrics showed TargetResponseTime increasing from 0.3 seconds to 2.4 seconds.
4. At 14:07:30, application logs reported only one healthy target and elevated response times.
5. At 14:09:00, the latency alarm changed from OK to ALARM after exceeding the configured threshold.

### Supported Findings

The evidence shows that a web-tier Auto Scaling group configuration change reduced available capacity from three instances to one instance. Shortly after the change, application response times increased significantly, logs confirmed only one healthy target remained, and the latency alarm triggered. The timing and correlation among CloudTrail, AWS Config, CloudWatch Metrics, CloudWatch Logs, and CloudWatch Alarms support the conclusion that the capacity reduction was the most likely cause of the website slowdown.

### Assumptions and Unresolved Questions

The available evidence does not identify a specific individual responsible for the change because the activity was performed through an assumed role session. Additional identity-provider logs, authentication records, or change-management documentation would be required to determine whether the change was performed manually, automatically, or as part of an authorized process. No approved change request was included in the evidence packet.

## Recommendation

I recommend that the cloud operations team verify the approved baseline configuration for the web-tier Auto Scaling group and review change-management records for the incident window. If the current configuration does not match approved settings, the team should evaluate whether a rollback or capacity correction is appropriate. Monitoring should continue to ensure latency returns to normal levels after any approved corrective action. All evidence should be preserved for future review.

## Escalation Notes

This incident should be escalated to the cloud operations team because a production configuration change occurred without an approved change request on file. Further review is needed to determine whether the change was authorized and whether additional corrective actions are required. The evidence does not support attribution to a specific individual and no personnel action should be taken based solely on the records reviewed.

## Lessons Learned

This week demonstrated the importance of correlating multiple evidence sources when investigating operational issues. Metrics show when performance changed, alarms show when thresholds were exceeded, logs provide application-level details, CloudTrail records management activity, and AWS Config documents configuration changes. Reviewing these sources together provides a more complete incident timeline and helps prevent unsupported conclusions. I also learned that attribution requires additional evidence and should not be assumed from a single audit record.

## Professional Vocabulary

**Monitoring:** The ongoing observation of system performance, availability, and health.

**Metric:** A numerical measurement collected over time, such as response time or CPU utilization.

**Log:** A timestamped record of actions, events, warnings, or errors generated by a system or application.

**Alarm:** A monitoring rule that triggers when a metric exceeds a defined threshold.

**CloudWatch:** AWS monitoring service used to collect metrics, logs, and alarms.

**CloudTrail:** AWS audit service that records API actions and management events.

**AWS Config:** AWS service that tracks resource configurations and configuration changes over time.

**Audit Trail:** A historical record of activity used to support investigations and compliance requirements.

**Configuration History:** A record showing how a resource's settings changed over time.

**Incident Timeline:** A chronological reconstruction of events related to an operational issue.

**Evidence Correlation:** The process of comparing and connecting information from multiple sources to determine what happened and when it occurred.
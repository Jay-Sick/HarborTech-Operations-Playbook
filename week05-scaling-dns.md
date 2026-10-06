# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary

TKT-2026-0005 addressed a seasonal traffic surge expected to significantly increase application demand. The current environment relies on a single EC2 instance and a single public endpoint, creating both a capacity limitation and a single point of failure. The ticket proposed using Auto Scaling, load balancing, and Route 53 failover capabilities to improve availability and support future growth.

## Client Impact

If traffic increases beyond the capacity of the current server, users may experience slow response times, application errors, or service outages. Because the environment depends on one EC2 instance and one endpoint, a server failure or endpoint outage could make the application unavailable to customers. These risks could negatively impact business operations, customer experience, and revenue during the seasonal promotion.

## Provided Ticket Evidence

- Previous promotion traffic reached 92% CPU utilization.
- Expected request volume is projected to increase significantly.
- Proposed Auto Scaling configuration: Minimum 2, Desired 2, Maximum 6.
- Proposed Application Load Balancer reporting two healthy targets.
- Proposed use of Route 53 DNS failover for improved availability.

This information was supplied in the ticket and was not personally verified in AWS.

## AWS Commands Used

### Auto Scaling Investigation

    aws autoscaling describe-auto-scaling-groups       --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}'       --output table

### Target Group Investigation

    aws elbv2 describe-target-groups       --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}'       --output table

### Route 53 Investigation

    aws route53 list-hosted-zones       --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}'       --output table

## AWS Evidence Collected

No Auto Scaling Groups were returned.

No target groups were returned.

No hosted zones were returned.

## Expected Results vs Actual Results

### Expected Results (Based on Ticket Evidence)

- Auto Scaling Group configured with Min 2, Desired 2, Max 6.
- Application Load Balancer with two healthy targets.
- Route 53 hosted zone configured with failover routing.
- Health checks supporting automatic failover.

### Actual Results (AWS Investigation)

- No Auto Scaling Groups found.
- No target groups found.
- No Route 53 hosted zones found.
- No AWS evidence supporting load balancing or DNS failover.

## Virtualization Connection

Virtualization allows compute resources to be created, replaced, and scaled through software rather than relying on a single physical server. Auto Scaling can automatically launch additional EC2 instances when demand increases and remove them when demand decreases. Load balancing distributes requests across multiple virtual servers, improving both performance and availability.

## Operational Analysis

The ticket evidence identifies a capacity and availability risk. Previous utilization reached 92% CPU on a single instance, and future traffic is expected to increase. However, the AWS investigation did not find any Auto Scaling Groups, target groups, or Route 53 hosted zones. Therefore, scaling, load balancing, health checks, and DNS failover could not be verified.

## Recommendation

I recommend implementing the proposed Auto Scaling configuration and an Application Load Balancer to reduce capacity and availability risks. Because no Route 53 configuration was verified, DNS failover should not be considered operational until hosted zones, health checks, failover records, and secondary endpoints have been configured and tested.

## Escalation Notes

As an intern, I am providing an operational recommendation only. Any production deployment involving Auto Scaling Groups, Application Load Balancers, Route 53 failover policies, health checks, or DNS changes should be reviewed and approved by the appropriate cloud operations team.

## Lessons Learned

This investigation demonstrated the importance of distinguishing ticket evidence from AWS evidence collected directly from the environment. Auto Scaling improves capacity, load balancing improves traffic distribution, and Route 53 supports DNS routing and failover.

## Professional Vocabulary

### Elasticity
The ability of a cloud environment to automatically add or remove resources as demand changes.

### Scalability
The ability of a system to handle increased workloads without major performance issues.

### Load Balancer
A service that distributes incoming traffic across multiple servers.

### Target Group
A collection of resources that receive traffic from a load balancer.

### Health Check
A monitoring process that determines whether a resource is functioning correctly.

### Auto Scaling Group
A service that automatically launches or terminates EC2 instances.

### Launch Template
A reusable configuration that defines how new EC2 instances are created.

### Desired Capacity
The number of EC2 instances an Auto Scaling Group attempts to maintain.

### Route 53
AWS DNS service used for routing and failover.

### Failover
The automatic redirection of traffic from an unavailable resource to a healthy backup resource.

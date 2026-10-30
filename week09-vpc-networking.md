# Week 9: VPC Networking and Troubleshooting

## HarborTech Ticket Summary

Ticket TKT-2026-0009 involved a web application hosted on an EC2 instance that users could not reach from the internet. The evidence showed that the instance was running, the web service was listening on TCP port 80, and the instance had a public IPv4 address. Security group and network ACL configurations appeared to allow the required traffic. However, a review of the subnet's associated route table revealed that only a local VPC route was present. The investigation focused on tracing the packet path from the internet client to the EC2 instance and identifying the earliest layer where the evidence no longer supported internet connectivity.

## Client Impact

Incorrect routing, subnet design, or traffic filtering can make an application unavailable even when the server and application are functioning correctly. If users cannot reach the application, business operations may be disrupted, customer access may be affected, and support costs may increase while teams troubleshoot the issue. Network connectivity failures can create downtime despite healthy compute resources.

## AWS Services Involved

- Amazon VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- CIDR Blocks
- Public and Private Subnets
- VPC Flow Logs

## Virtualization Connection

Amazon VPC uses software-defined networking to create isolated virtual networks in the cloud. Instead of configuring physical switches and routers, administrators configure CIDR blocks, subnets, route tables, gateways, and security controls through AWS. Even though the infrastructure is virtualized, packets still require valid addressing, routing, gateway connectivity, and security permissions to successfully travel from a source to a destination.

## Evidence Reviewed

- Required traffic flow: Internet client → TCP 80 → EC2 web server
- VPC ID: vpc-0a91
- VPC CIDR: 10.42.0.0/16
- Subnet ID: subnet-0a91
- EC2 instance state: Running
- Public IPv4 address: 198.51.100.24
- Application status: Listening on TCP port 80
- Route table: rtb-0a91
- Route table entry:
  - 10.42.0.0/16 → local (active)
- Internet Gateway:
  - igw-0a91 attached to vpc-0a91
- Security Group:
  - Inbound TCP 80 from 0.0.0.0/0 allowed
- Network ACL:
  - Inbound TCP 80 allowed
  - Outbound TCP 1024-65535 allowed
- Connectivity test results:
  - Not provided in the evidence packet
- VPC Flow Logs:
  - Not provided in the evidence packet

## Operational Analysis

The investigation followed the expected traffic path from an internet client to the EC2 web server. The instance was confirmed to be running and listening on the required application port. Both the security group and network ACL contained rules that would allow the required HTTP traffic.

The first unsupported assumption was that the subnet was truly a public subnet. While the instance had a public IPv4 address and the VPC had an attached Internet Gateway, the associated route table only contained the route:

- 10.42.0.0/16 → local

No default route such as:

- 0.0.0.0/0 → igw-0a91

was shown in the evidence.

Without a default route to the Internet Gateway, there is no proven path for internet traffic to reach the subnet. Because routing is evaluated before security controls can allow traffic to a resource, the route table is the most likely blocking layer. The security group and network ACL findings help rule out an obvious firewall restriction but do not prove reachability because packets must first reach the subnet through a valid route.

## Recommendation

The recommended next action is to verify the complete route table configuration associated with subnet-0a91 and determine whether a default route to the Internet Gateway exists.

Specifically, confirm whether the following route is present:

- 0.0.0.0/0 → igw-0a91

This is the smallest supported action because it validates the routing configuration without making unnecessary changes to security groups, network ACLs, or other network components.

After verification, perform an external HTTP connectivity test to 198.51.100.24 on TCP port 80. If the route exists and is correctly associated, the expected result is a successful HTTP response from the application.

## Escalation Notes

Any modification to production route tables, including adding, removing, or changing a default route, requires approval and escalation beyond intern authority.

Changes requiring escalation include:

- Adding a 0.0.0.0/0 route
- Changing route-table associations
- Modifying gateway configurations
- Altering production security groups
- Altering production network ACLs
- Changing VPC network architecture

Inspection and documentation are within intern authority, but production network changes must be approved by the appropriate cloud operations or network administration team.

## Lessons Learned

This week's exercise demonstrated that network troubleshooting should follow the complete traffic path rather than focusing immediately on firewall settings. A subnet labeled as public is not necessarily internet reachable. Public reachability requires proper addressing, route-table configuration, gateway connectivity, and security controls. Security groups and network ACLs may allow traffic, but they cannot compensate for missing routes. Effective troubleshooting depends on verifying each network layer with evidence and identifying the earliest point where the intended path is not supported.

## Professional Vocabulary

**VPC**  
A logically isolated virtual network in AWS where resources, routes, and security controls are managed.

**Subnet**  
A smaller IP address range within a VPC that contains resources and is associated with a route table.

**Route Table**  
A collection of routing rules that determines where traffic is sent from a subnet.

**Internet Gateway**  
A VPC component that enables communication between resources and the public internet when routing is correctly configured.

**NAT Gateway**  
A managed AWS service that allows private-subnet resources to access the internet for outbound connections while preventing unsolicited inbound internet traffic.

**Security Group**  
A stateful virtual firewall attached to AWS resources that controls allowed inbound and outbound traffic.

**Network ACL**  
A stateless subnet-level firewall that evaluates inbound and outbound traffic using ordered rules.

**CIDR**  
Classless Inter-Domain Routing notation used to define IP address ranges such as 10.42.0.0/16.

**Public Subnet**  
A subnet whose associated route table contains a route to an Internet Gateway, allowing internet connectivity.

**Private Subnet**  
A subnet that does not have a direct route to an Internet Gateway and typically uses a NAT Gateway for outbound internet access.
# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary

Ticket **TKT-2026-0006** focused on evaluating whether two small workloads currently running on a single EC2 instance actually require a continuously running server. The environment contains a daily reporting task that runs once each morning and a small REST API that receives intermittent requests throughout the day.

The compute-model problem is that the EC2 instance remains idle for long periods while HarborTech and the client still maintain the operating system, application runtime, security updates, monitoring, and server administration. The goal is to determine whether a virtual machine, container, or serverless approach provides the best operational fit for each workload.

## Client Impact

Using an inefficient compute model can negatively affect several operational areas:

**Cost:** Running a virtual machine continuously for workloads that execute only occasionally can increase infrastructure costs because resources remain allocated even when no work is being performed.

**Reliability:** Poorly matched compute resources may introduce unnecessary maintenance events, operating-system issues, or configuration drift that can impact availability.

**Scalability:** A fixed server may not automatically scale with demand. Serverless and container-based solutions can often scale more efficiently based on workload requirements.

**Operational Effort:** Maintaining servers requires operating-system patching, security management, backups, monitoring, troubleshooting, and runtime maintenance.

## AWS Services Involved

- Containers
- Serverless Computing
- AWS Lambda
- Amazon API Gateway
- AWS Step Functions
- IAM Execution Roles
- Event-Driven Computing
- VPC-Connected Lambda
- Monitoring and Logs

## Virtualization Connection

Traditional virtual machines abstract physical hardware but still require administrators to maintain a guest operating system. Containers provide a higher level of abstraction by packaging the application and its dependencies while sharing the host operating-system kernel. Serverless computing provides an even higher level of abstraction where AWS manages the infrastructure, operating system, scaling, and execution environment.

## Evidence Reviewed

- Daily reporting task runs once each morning.
- Reporting task completes quickly.
- Reporting task maintains no local state between executions.
- Current EC2 instance remains idle for significant periods.
- REST API receives intermittent requests.
- Workload appears event-driven.
- No evidence indicated a requirement for persistent local storage.
- No evidence indicated a requirement for custom operating-system modifications.

## Operational Analysis

The evidence indicates that the current server-based approach may be larger than necessary for the workloads being executed.

The daily reporting task is a strong candidate for AWS Lambda because it runs on a schedule, completes quickly, and does not require persistent local state.

The REST API appears well suited for an API-driven serverless architecture using API Gateway and Lambda because it receives intermittent traffic and does not require a continuously running server based on the available evidence.

## Recommendation

### Reporting Task

**Recommended Model:** AWS Lambda

Responsibilities moved to AWS include infrastructure management, operating-system maintenance, scaling, and execution environment availability.

Responsibilities remaining with HarborTech include report code, scheduling configuration, IAM permissions, monitoring, and report validation.

### REST API

**Recommended Model:** Amazon API Gateway with AWS Lambda

Responsibilities moved to AWS include server infrastructure, runtime hosting, scaling, and availability management.

Responsibilities remaining with HarborTech include API code, business logic, security controls, integrations, testing, and monitoring.

## Escalation Notes

The following items require approval before implementation:

- Production architecture changes
- Migration from EC2 to Lambda
- API deployment modifications
- Network and security configuration changes
- IAM permission changes
- Database integration updates

## Lessons Learned

This week demonstrated that selecting a compute model should be based on workload behavior rather than service popularity. Event-driven workloads are often strong candidates for serverless execution, while virtual machines and containers remain valuable when additional control or portability is required.

## Professional Vocabulary

**Container:** A packaged application environment that includes the application and its dependencies.

**Serverless Computing:** A cloud execution model where the provider manages infrastructure while the customer focuses on code.

**AWS Lambda:** An AWS service that runs code in response to events without requiring server management.

**API Gateway:** A service that receives and manages API requests and routes them to backend services.

**Execution Role:** An IAM role that grants AWS services permission to access AWS resources.

**Event Trigger:** An action that starts workload execution.

**Step Functions:** An AWS service used to coordinate multiple tasks into a workflow.

**VPC-Connected Lambda:** A Lambda function configured to access resources in a VPC.

**Runtime:** The software environment that executes application code.

**Abstraction:** Hiding lower-level infrastructure details so users can focus on application logic.

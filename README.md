# aws-security-audit-and-ha-validation
An end-to-end AWS security audit, vulnerability remediation, and high-availability validation using ALB, Auto Scaling, and CloudWatch metrics

# AWS Security Hardening, Infrastructure Audit, and High-Availability Validation

## Project Overview
This project simulates a real-world AWS security assessment by intentionally introducing common cloud security misconfigurations, performing a structured audit to identify unauthorized access risks, and implementing remediation using AWS security best practices. Additionally, this project diagnoses and eliminates infrastructure elasticity bottlenecks by implementing and validating an automated scaling architecture under synthetic load, aligning the entire environment with the pillars of the AWS Well-Architected Framework.

## Business Scenario
An organization requested an external infrastructure security audit after concerns that years of rapid deployment cycles had introduced excessive IAM permissions, publicly exposed storage buckets, and overly permissive firewall rules. Furthermore, the application suffered from complete performance degradation during unpredicted traffic surges. 

The objective of this engagement was to map out security risks, execute strict remediation steps without disrupting the baseline environment, introduce automated elasticity to handle traffic bottlenecks, and validate the system's real-time recovery response using heavy performance load testing.

---

## Architecture & Service Diagrams

### 1. AWS Well-Architected Infrastructure Diagram
*This dynamic multi-AZ visual trace models network containment boundaries, horizontal scaling groups, and traffic routing tiers.*

```mermaid
graph TB
    %% Styling and Theme definitions for modern AWS theme
    classDef Internet fill:#f5f5f5,stroke:#232F3E,stroke-width:2px;
    classDef Network fill:none,stroke:#7AA116,stroke-width:2px,stroke-dasharray: 5 5;
    classDef ALB fill:#8C4FFF,stroke:#ffffff,stroke-width:2px,font-weight:bold,color:#ffffff;
    classDef Subnet fill:none,stroke:#00A1C9,stroke-width:2px;
    classDef ASG fill:none,stroke:#FF9900,stroke-width:2px,stroke-dasharray: 5 5;
    classDef Compute fill:#FF9900,stroke:#ffffff,stroke-width:1px,color:#ffffff;
    classDef Storage fill:#3F8624,stroke:#ffffff,stroke-width:1px,color:#ffffff;

    User([Public Traffic Ingress]) ::: Internet
    IGW[Internet Gateway] ::: Network
    
    subgraph VPC [Amazon VPC Context - 10.0.0.0/16]
        direction TB
        
        Proxy[Application Load Balancer <br> Target Group Tier] ::: ALB
        
        subgraph AZ_A [Availability Zone A]
            direction TB
            subgraph Subnet_A [Public Subnet A - 10.0.1.0/24]
                direction TB
                subgraph ASG_Group [ticketing-app-asg]
                    Node_A[EC2 Instance <br> Baseline Node] ::: Compute
                end
            end
        end

        subgraph AZ_B [Availability Zone B]
            direction TB
            subgraph Subnet_B [Public Subnet B - 10.0.2.0/24]
                direction TB
                Node_B[EC2 Instance <br> Scale-Out Target] ::: Compute
            end
        end
    end

    subgraph Security_Audit_Layer [Out-of-Band Security Assessment]
        Analyzer[IAM Access Analyzer] ::: Storage
        S3[Hardened S3 Bucket] ::: Storage
    end

    subgraph Telemetry [CloudWatch Observability Framework]
        Metrics[CloudWatch Telemetry Engine] ::: ALB
        Alarm[Dynamic Scale Out Alarm Trigger] ::: Compute
    end

    %% Network Routing Traces
    User --> IGW
    IGW --> Proxy
    Proxy -->|Layer-7 Route| Node_A
    Proxy -->|Scale-Out Vector| Node_B
    
    %% Scaling Triggers
    Node_A -.->|CPU > 50% Threshold| Metrics
    Metrics -->|Trips State Change| Alarm
    Alarm -->|Orchestrates Launch| Node_B

    %% Verification Proof Mappings
    click Proxy "[https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/02-load-balancer-created.png](https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/02-load-balancer-created.png)" "View Load Balancer Proof"
    click Node_A "[https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/05-single-instance-running.png](https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/05-single-instance-running.png)" "View Baseline Instance"
    click Node_B "[https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/10-asg-scale-out-instances.png](https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/10-asg-scale-out-instances.png)" "View Scale Out Event"
    click Metrics "[https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/08-cpu-utilization-spike.png](https://github.com/your-username/aws-security-audit-and-ha-validation/blob/main/08-cpu-utilization-spike.png)" "View CloudWatch Pyramid Spike"
 
graph LR
    subgraph Identity_Governance [Security & Audit Pillar]
        IAM[AWS IAM <br> Access Rules] --->|Scans Privilege Blocks| AA[IAM Access Analyzer]
        AA --->|Global Block Applied| S3[Amazon S3 <br> Object Storage]
    end

    subgraph Elastic_Core [Compute & Automation Layer]
        VPC[Amazon VPC] ---> EC2[Amazon EC2 Compute Node]
        EC2 <--->|Metrics Stream| CW[Amazon CloudWatch Logging]
        CW --->|Orchestration Request| ASG[EC2 Auto Scaling Groups]
    end
    
    style IAM fill:#CD2264,color:#ffffff,stroke:none
    style AA fill:#CD2264,color:#ffffff,stroke:none
    style S3 fill:#3F8624,color:#ffffff,stroke:none
    style VPC fill:#7AA116,color:#ffffff,stroke:none
    style EC2 fill:#FF9900,color:#ffffff,stroke:none
    style CW fill:#4D27AA,color:#ffffff,stroke:none
    style ASG fill:#FF9900,color:#ffffff,stroke:none

## Services Utilized & Engineering Purpose

| Service | Modern AWS Icon Framework | Strategic Project Purpose |
| :--- | :--- | :--- |
| **Amazon VPC** | Virtual Private Cloud | Isolated, custom-routed networking layer to prevent default-subnet exposures. |
| **Amazon EC2** | Elastic Compute Cloud | Hosted scalable compute infrastructure utilized to model application workloads. |
| **Application Load Balancer** | Elastic Load Balancing | Distributed user ingress across multiple target availability zones to eliminate single points of failure. |
| **Amazon S3** | Simple Storage Service | Object storage layer evaluated for resource access policies and public leakage vectors. |
| **AWS IAM** | Identity & Access Management | Granular identity management utilized to build, evaluate, and enforce least-privilege roles. |
| **IAM Access Analyzer** | IAM Feature | Automated scanning mechanism used to detect cross-account or public entry paths. |
| **AWS CloudWatch** | CloudWatch Metrics/Alarms | Centralized log collection, performance metric gathering, and alarm threshold tracking. |
| **Auto Scaling Groups** | EC2 Auto Scaling | Automated compute orchestration infrastructure to dynamic scale resources based on load metrics. |

---

## Security Risks Identified & Audit Log

| Finding | Risk Description | Remediation Implemented | Verification Artifact |
| :--- | :--- | :--- | :--- |
| **AdministratorAccess** | Excessive privileges assigned to standard developer account allowing full lateral account takeover. | Stripped wildcard policy; applied strict, scoped role mapping. | `05-iam-access-analyzer.png` |
| **Public S3 Bucket** | Corporate asset exposure and data leak vulnerability due to disabled public access blocks. | Enabled AWS Block Public Access globally at the bucket level. | `06-s3-public-blocked.png` |
| **SSH Open to Internet** | Global inbound port 22 access (0.0.0.0/0) welcoming external brute-force vectors. | Scoped inbound security group rules tightly to authorized IP vectors only. | `07-security-group-hardened.png` |

The Bottleneck
During initial baseline load assessments, testing confirmed that while the application was reachable and reading correctly (07-application-running.png), the application stack experienced complete connection timeouts and session drops under synthetic traffic spikes.

Deep-dive monitoring of baseline telemetry (06-baseline-cloudwatch-metrics.png) revealed that a single compute node was forced to maintain 100% CPU utilization without any mechanism to distribute load or spawn parallel compute capacity. This architectural choke point represented an unmitigated operational risk, exposing the business to prolonged downtime under unexpected user surges.

Resolution & Elastic Validation
To resolve this infrastructural limit, an Application Load Balancer and an Auto Scaling Group were deployed. A target tracking policy was engineered with an aggressive threshold set at 50% average CPU Utilization monitored over 1-minute intervals via CloudWatch Metrics.

To validate the architecture, the stress utility was introduced directly into the instance environment to emulate an emergency compute crunch: stress --cpu 4 --timeout 600

Validation Results

Metric Spike Trace: CloudWatch captured the breach immediately, illustrating a sharp upward trajectory peaking at 100% capacity in a distinct pyramid pattern (08-cpu-utilization-spike.png).

Horizontal Scaling Launch: The scaling policy evaluated the anomaly and automatically triggered an orchestration action, provisioning a parallel backup EC2 instance into service within 180 seconds to absorb the load (10-asg-scale-out-instances.png).

Orchestration Log Auditing: The Auto Scaling Group activity records confirmed successful infrastructure provisioning and health check execution under load (11-asg-activity-history.png).

Skills Demonstrated

IAM Administration & Least-Privilege Design Patterns

Custom Network Layer Design (VPC, Subnetting, Ingress/Egress Routing)

Public Cloud Security Auditing & Structural Governance

Attack Surface Mapping and Vulnerability Remediation

Distributed Systems High-Availability Engineering

Infrastructure Performance Testing & Load Simulation

Cloud Monitoring, Metrics Aggregation, and Observability

Lessons Learned

Continuous Auditing Over Reactive Remediation: Security postures change rapidly during active deployment cycles. Utilizing automated scanning loops (like IAM Access Analyzer) continuously prevents configuration drift before vulnerabilities reach a production scope.

Blast Radii Reduction via Principle of Least Privilege: Restricting developer and service roles to precisely what they require ensures that credential leaks or system compromises do not grant an attacker full control of the cloud environment.

Security Must Complement High Availability: Hardening the security architecture of an environment is meaningless if the application goes offline due to basic resource constraints. True cloud engineering treats scalability as a core tenant of operational security.

Validation via Failure Injection: You don't truly know if your monitoring or scaling policies work until you intentionally try to break them. Simulating live production failures via stress utilities is crucial to verifying automation boundaries before an actual emergency incident occurs.

Business Impact

Drastically Reduced Attack Surface: Eliminated open entry points to compute nodes and sealed storage layers containing proprietary data.

Hardened Security Governance: Aligned corporate permission matrices directly with AWS security best practices and compliance benchmarks.

Guaranteed Business Continuity: Proven via active load simulation that security modifications did not impact application elasticity, ensuring optimal uptime and automated self-healing during high-traffic business events.

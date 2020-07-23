# AWS Certified SAA-C02 Course
Notes for Adrian Cantrill course https://learn.cantrill.io/

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Virtual Private Cloud Basics](#virtual-private-cloud-basics)
  * [VPC Considerations](#vpc-considerations)
  * [VPC Sizing](#vpc-sizing)
  * [Custom VPC](#custom-vpc)

# Virtual Private Cloud Basics

### VPC Considerations

- What **size** should the VPC be..
- Are there any Networks **we can't use** ...
- VPC's, Cloud, On-premises, Partners & Vendors
- Try to predict the **future**..
- VPC **Structure** - Tiers & Resiliency (Availability) Zones
- VPC minimum **/28** (16 IP), maximum **/16** (65456 IPs)
- **Avoid common ranges** - avoid future issues

### VPC Sizing

| VPC Size    | Netmask | Subnet Size | Hosts/Subnet | Subnets/VPC | Total IPs |
| ----------- | ------- | ----------- | ------------ | ----------- | --------- |
| Micro       | /24     | /27         | 27           | 8           | 216       |
| Small       | /21     | /24         | 251          | 8           | 2008      |
| Medium      | /19     | /22         | 1019         | 8           | 8152      |
| Large       | /18     | /21         | 2043         | 8           | 16344     |
| Extra Large | /16     | /20         | 4091         | 16          | 65456     |

- Subnet Calculator : https://www.site24x7.com/tools/ipv4-subnetcalculator.html
- How many **subnets** will you need?
- How many **IPs total**? How many **per subnet**?

### Custom VPC

- Regional Service - All AZs in the region
- Isolated network
- Nothing **IN** or **OUT** without explicit configuration
- Flexible configuration - simple or multi-tier
- Hybrid Networking - other cloud & on-premises
- **Default** or **Dedicated Tenancy!**
- IPv4 Private CIDR Blocks & Public IPs
- 1 Primary Private IPv4 CIDR Block
- ...Min **/28** (16 IP) Max **/16** (65536 IP)
- Optional secondary IPv4 Blocks
- Optional single **assigned** IPv6 **/56** CIDR Block

### DNS in a VPC

- Provided by R53
- VPC **Base IP +2** Address, for example if the VPC IP is 10.0.0.0 then the DNS IP will be 10.0.0.2
- **enableDNSHostnames** - gives instances DNS Names
- **enableDNSSupport** - enables DNS resolution in VPC

### VPC Subnets

- **AZ Resilient**
- A subnetwork of a VPC - **within a particular AZ**
- 1 Subnet => 1 AZ, 1 AZ => 0+ Subnets
- IPv4 CIDR is a subset of the VPC CIDR
- Cannot overlap with other subnets
- Optional IPv6 CIDR (/64 subset of the /56 VPC - space for 256)
- Subnets can communicate with other subnets in the VPC

### Subnet IP Addressing

- Reserved IP addresses (5 in total)
- For example: 10.16.16.0/20 (10.16.16.0 => 10.16.31.255)
- **Network** Address (10.16.16.0)
- **Network +1** (10.16.16.1) - VPC Router
- **Network +2** (10.16.16.2) - Reserved (DNS*)
- **Network +3** (10.16.16.3) - Reserved Future Use
- **Broadcast** Address 10.16.31.255 (Last IP in subnet)

### VPC Router

- Every VPC has a VPC Router - Highly available
- In every subnet .. **network +1** address
- Routes traffic between subnets
- Controlled by **route tables** each subnet has one
- A VPC has a **Main** route table - subnet default

### Internet Gateway (IGW)

- **Region resilient** gateway attached to a VPC
- 1 VPC = 0 or 1 IGW, 1 IGW = 0 or 1 VPC
- Runs from within the AWS Public Zone
- Gateways traffic between the VPC and the Internet or AWS Public Zone (S3..SQS..SNS..etc)
- Managed - AWS handles performance

### Using an IGW

1. Create IGW
2. Attach IGW to VPC
3. Create custom RT
4. Associate RT
5. Default Routes => IGW
6. Subnet allocate IPv4

### Bastion Host / Jumpbox

- Bastion Host = Jumpbox
- An instance in a public subnet
- Incoming management connections arrive there
- Then access internal VPC resources
- Often the only way IN to a VPC

### Network Access Control List (NACL)

- **Stateless** - INITIATION and RESPONSE seen as different
- **Only** impacts data **crossing subnet border**
- Can **EXPLICITLY ALLOW** and **DENY**
- IPs/Networks, Ports & Protocols - **no logical resources**
- NACLs cannot be assigned TO AWS resources.. only subnets
- Use WITH **Security Groups** to add explicit **DENY** (Bad IPs/Nets)
- One subnet = One NACL at a time

### Security Groups (SG)

- **Stateful** - TRAFFIC and RESPONSE = **Same Rule**
- **Security Groups** can filter based on **AWS Logical resources**...
- ... Resources, other Security Groups and even themselves
- Implicit Deny and Explicit Allow
- .. **NO EXPLICIT DENY**

### SGs vs NACL

- NACLs on subnet for any products which don't work with SG's e.g. NAT Gateways
- NACLs when adding explicit DENY (bad IP's, bad actors)
- SG as the default **almost everywhere**

### What is NAT?

- Network Address Translation (**NAT**)
- A set of processes - remapping SRC or DST IPs
- **IP masquerading** - hiding CIDR Blocks behind one IP
- Private IPv4 Addresses running out
- Gives Private CIDR range **outgoing** internet access

### NAT Gateways

- Runs from a **public subnet**
- Uses **Elastic IPs** (Static IPv4 Public)
- **AZ resilient Service** (HA in that AZ)
- For region resilience - **NATGW in each AZ ...**
- .. RT in for each AZ with that NATGW as target
- Managed, scales to 45 Gbps, $ Duration & Data Volume

### NAT Instance vs NAT Gateway

| Attribute | NAT gateway | NAT instance |
| --------- | ----------- | ------------ |
| Availability | Highly available. NAT gateways in each Availability Zone are implemented with redundancy. Create a NAT gateway in each Availability Zone to ensure zone-independent architecture.| Use a script to manage failover between instances.|
| Bandwidth | Can scale up to 45 Gbps. | Depends on the bandwidth of the instance type |
| Maintenance | Managed by AWS. You do not need to perform any maintenance. | Managed by you, for example, by installing software updates or operating system patches on the instance. |
| Performance | Software is optimized for handling NAT traffic. | A generic Amazon Linux AMI that's configured to perform NAT. |
| Cost | Charged depending on the number of NAT gateways you use, duration of usage, and amount of data you send through the NAT gateways. | Charged depending on the number of NAT instances that you use, duration of usage, and instance type and size. |
| Type and size | Uniform offering; you don't need to decide on the type or size. | Choose a suitable instance type and size, according to your predicted workload. | 
| Security groups | Cannot be associated with a NAT gateway. You can associate security groups with your resources behind the NAT gateway to control inbound and outbound traffic. | Associate with your NAT instance and the resources behind your NAT instance to control inbound and outbound traffic. | 
| Network ACLs | Use a network ACL to control the traffic to and from the subnet in which your NAT gateway resides. | Use a network ACL to control the traffic to and from the subnet in which your NAT instance resides. | 
| Flow logs | Use flow logs to capture the traffic. | Use flow logs to capture the traffic. | 
| Port forwarding | Not supported. | Manually customize the configuration to support port forwarding. | 
| Bastion servers | Not supported. | Use as a bastion server. |

### What about IPv6?

- NAT isn't required for IPv6
- All IPv6 addresses in AWS are publicly routable
- The Internet Gateway works with ALL IPv6 IPs directly
- NAT Gateways **don't work with IPv6**
- ::/0 Route + IGW for bi-directional connectivity
- ::/0 Route + Egress-Only Internet Gateway - Outbound Only

# Elastic Compute Cloud (EC2) Basics

### EC2 Architecture

- EC2 Instance are **virtual machines** (OS+Reseources)
- EC2 Instances run on **EC2 Hosts**
- **Shared** Hosts or **Dedicated** Hosts
- **Hosts = 1 AZ** - AZ Fails, Host Fails, Instances Fail

### What's EC2 Good for?

- Traditional **OS+Application** Compute
- **Long-Running** Compute
- **Server** style applications ...
- .. either **burst** or **steady-state** load
- **Monolithic** application stacks
- **Migrated** application workloads or **Disaster Recovery**

### EC2 Instance Types

- **Raw** CPU, Memory, Local Storage Capacity & Type
- **Resource Ratios**
- **Storage** and **Data** Network **Bandwidth**
- System Architecture / Vendor
- Additional Features and Capabilities

### EC2 Categories

- **General Purpose** - Default - Diverse workloads, equal resource ratio.
- **Compute Optimized** - Media Processing, HPC, Scientific Modelling, gaming, Machine Learning.
- **Memory Optimized** - Processing large in-memory datasets, some database workloads.
- **Accelerated Computing** - Hardware GPU, field programmable gate arrays (FPGAs).
- **Storage Optimized** - Sequential and Random IO - scale-out transactional databases, data warehousing, Elasticsearch, analytics workloads.

### Key Terms Storage Refresher 

- **Direct** (local) attached Storage - Storage on the EC2 Host
- **Network** attached Storage - Volumes delivered over the network (EBS)
- **Ephemeral** Storage - Temporary Storage
- **Persistent** Storage - Permanent storage - lives on past the lifetime of the instance

### Key Terms Storage Refresher - Part 2

- **Block** Storage - **Volume** presented to the **OS** as a collection of blocks... no structure provided. **Mountable**. **Bootable**.
- **File** Storage - Presented as a file share ... has structure. **Mountable**. **NOT Bootable**.
- **Object** Storage - collection of objects, flat. **Not mountable**. **Not bootable**.

### EBS & Volume Types

- Volumes created in an AZ, **isolated in that AZ**
- AZ fails - Volume impacted.. **Snapshot help**
- **Highly available and resilient in that AZ**
- Generally one volume <-> 1 instance (..but multi-attach)
- GB/month fee regardless of instance state...
- EBS MAX 80k IOPS (**Instance**), 64k (**Vol**) (**io1**)
- ..MAX 2375 MB/s (**Instance**), 1000 MiB/s (**Vol**) (**io1**)

### Instance Store Volumes

- Local on **EC2 Host**
- **Block Storage** Devices
- Physically connected to **one EC2 Host**
- Instances **on that host** can access them
- Lost on instance **move**, **resize** or **hardware failure**
- Highest storage performance in AWS
- You pay for it anyway - nncluded in instance price
- **ATTACHED AT LAUNCH**
- **TEMPORARY**

### When to use EBS

- **Highly Available** and **Reliable** storage
- **Persist independently** from the EC2 Instance
- Clusters - **Multi-Attach** feature of io1
- Region Resilient **Backups**
- Require up to **64000 IOPS** and **1000 MiB/s** per volume
- Require up to **80000 IOPS** and **2375 MB/s** per instance

### When to use Instance Store

- **Value** - Included in instance cost
- **More than 80000 IOPS & 2375 MB/s**
- **Temp Storage** volumes
- **Stateless** services
- Rigid lifecycle link .. **storage <-> Instance**

### EBS Snapshots

- Snapshots are incremental volume copies to **S3**
- The first is a **full copy** of 'data' on the volume
- Future snaps are **incremental**
- Volumes can be created (restored) from snapshots
- Snapshots can be copied to another region

### EBS Snapshots/Volume Performance

- New EBS volume = **full performance immediately**
- **Snaps restore lazily** - fetched gradually
- Requested blocks are fetched immediately
- Force a read of all data immediately ...
- Fast Snapshot Restore (**FSR**) - Immediate restore
- .. up to **50** snaps per region. Set on the **Snap & AZ**

### EBS Encryption

- Accounts can be set to **encrypt by default** - default CMK
- Otherwise choose a CMK to use.
- Each volume uses **1 unique DEK**.
- Snapshots & future volumes use the **same DEK**.
- Can't change a volume to NOT be encrypted.
- OS isn't aware of the encryption ... no performance loss.

### EC2 Network & DNS Architecture

- **Primary ENI (Elastic Network Interfaces)**
  - Example: Mac Address - 00:0d:83:b1:c0:8e
  - Primary IPv4 Private IP => 10.16.0.10 => ip-10-16-0-10.ec2.internal
  - 0 or more secondary IPs
  - 0 or 1 Public IPv4 Address => 3.89.7.136 => ec2-3-89-7-136.compute-1.amazonaws.com
  - 1 elastic IP per private IPv4 address - Removes the Public IPv4 - Replaces with the Elastic IP
  - 0 or more IPv6 addresses
  - Security Groups
  - Source/Destination Check
- **Secondary ENI's**
  - As above

### EC2 Network & DNS Architecture

- Secondary ENI + MAC = **Licensing**
- Multi-homed (subnets) Management & Data
- Different Security Groups - **multiple interfaces**
- OS - **DOESN'T see public IPv4**
- IPv4 Public IPs are **Dynamic** .. Stop & Start = **Change**
- Public DNS = **private IP in VPC**, public IP everywhere else

### Amazon Machine Image (AMI)

- AMI's can be used to **launch EC2** instance
- **AWS** or **Community** Provided
- Marketplace (can include **commercial software**)
- **Regional** .. **unique ID e.g. ami-0a887e401f7654935
- Permissions (Public, Your Account, Specific Accounts)
- You can create an AMI from an EC2 instance you want to template

### AMI Exam Tips

- AMI = **One Region**, only works in that one region
- **AMI Baking** .. creating an AMI from a configured instance + application
- An AMI **can't be edited** .. launch instance, update configuration and make a new AMI
- Can be copied **between regions** (includes its snapshots)
- Remember permissions .. **default = your account**

### Instance Pricing Models

- On-Demand Instances
- Spot Instances
- Reserved Instances
- Dedicated Hosts

### On-Demand Instances

- Instances have an **hourly rate**
- Billed in **seconds** (**60s** minimum) or **Hourly**
- **Default** Pricing Model
- No **long-term** commitments or **upfront** payments
- **New** or **uncertain** application requirements
- **Short-term**, **spiky**, or **unpredictable** workloads which **can't tolerate any disruption**

### Spot Instances

- Spot pricing offers up to **90%** off vs On-Demand
- A **spot price** is set by EC2 - based on **spare capacity**
- You can specify a **maximum price** you'll pay
- If spot price goes above yours - **instances terminate**
- Applications that have flexible start and end times
- Apps which only make sense at low cost
- Apps which **can tolerate failure** and continue later

### Reserved Instances

- Up to 75% off vs On-demand - **for a commitment**
- 1 or 3 years, All Upfront, Partial Upfront, No Upfront
- Reserved in **region**, or **AZ** with capacity **reservation**
- Scheduled Reservations
- Known steady state usage
- Lowest cost for apps which can't handle disruption
- Need reserved capacity ..

### Vertical Scaling

- Each resize requires a reboot - **disruption**
- Large instances often carry a **$ premium**
- There is an upper cap on performance - **instance size**
- **No application modification** required
- Works for ALL applications - **even Monoliths**

### Horizontal Scaling

- Sessions, Sessions, Sessions
- Requires application support OR **off-host sessions**
- **No disruption** when scaling
- **No real limits** to scaling
- Often less expensive - **no large instance premium**
- More granular ..

# Containers & ECS

### Container Key Concepts

- **Dockerfiles** are used to **build images**
- Portable - self-contained, always run as expected
- Lightweight - Parent OS used, **fs layers are shared**
- Container only runs the application & environment it needs
- Provides much of the isolation VM's do
- Ports are '**exposed**' to the host and beyond...
- Application stacks can be multi-container ...

### ECS Concepts

- **Container Definition** - Image & Ports
- **Task Definition** - Security (Task Role), Containers(s), Resources
- **Task Role** - IAM Role which the TASK assumes
- **Service** - How many copies, HA, Restarts

### EC2 vs ECS (EC2) vs Fargate

- If you use containers ... **ECS**
- **Large** workload - **price** conscious - **EC2 Mode**
- **Large** workload - **overhead** conscious - **Fargate**
- **Small** / **Burst** workloads - **Fargate**
- **Batch** / **Periodic** workloads - **Fargate**

# Advanced EC2

### EC2 Bootstrapping

- Bootstrapping allows **EC2 Build Automation**
- User Data - Accessed via the meta-data IP
- **http://169.254.169.254/latest/user-data**
- Anything in User Data is **executed** by the **instance OS**
- **ONLY on Launch**
- EC2 doesn't interpret, the OS needs to understand the User Data

### User Data Key Points

- It's **opaque** to EC2 .. its just a **block of data**
- It's **NOT** secure - don't use it for passwords or long term credentials (ideally)
- User data is limited to 16 KB in size
- Can be modified when instance stopped
- But **only executed once at launch**

### EC2 Instance Role

- Credentials are inside meta-data
- iam/security-credentials/**role-name**
- Automatically rotated - Always valid
- Should always be used rather than adding access keys into instance
- CLI tools will use ROLE credentials automatically

### SSM Parameter Store

- Storage for **configuration** & **secrets**
- String, StringList & SecureString
- License codes, Database Strings, Full Configs & Passwords
- Hierarchies & Versioning
- Plaintext and Ciphertext
- Public Parameters - **Latest AMIs per region**

### EC2 Placement Groups

- **Cluster** - Pack instances close together
- **Spread** - Keep instances separated
- **Partition** - groups of intances spread apart

### Cluster Placement Groups

- Can't span AZs - **ONE AZ ONLY**
- Can span VPC peers - but impacts performance
- Requires a supported instance type
- Use the same type of instance (**not mandatory**)
- Launch at the same time (**not mandatory**)
- **10 Gbps single stream performance**
- Use cases: **Performance**, **fast speeds**, **low latency**

### Spread Placement Groups

- Provides infraestructure isolation - each **INSTANCE** run from a different rack
- **7 Instances per AZ** (HARD LIMIT)
- Not supported for Dedicated Instances or Hosts
- Use Case: Small number of critical instances that need to be kept separated from each other

### Partition Placement Groups

- **7 Partitions per AZ**
- Instances can be placed in **a specific partition**
- ...or auto placed
- Partition placement groups are not supported for Dedicated Hosts
- Great for HDFS, HBase, and Cassandra

### Links Placement Groups

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html

### EC2 Dedicated Hosts

- EC2 Host **dedicated to you**
- Specific family e.g. a1, c5, m5
- **No instance charges** ... you pay for the host
- On-Demand & Reserved Options available
- Host hardware has **physical sockets and cores**

### Limitations & Features

- **AMI Limits** - RHEL, SUSE Linux, and Windows AMIs aren't supported
- **Amazon RDS** instances are not supported
- **Placement groups** are not supported for Dedicated Hosts
- Hosts can be shared with other ORG Accounts...**RAM product (Resource Access Manager)**

### Enhanced Networking

- Uses **SR-IOV** - NIC is virtualization aware
- No charge - available on most EC2 Types
- **Higher I/O** & **Lower Host CPU Usage**
- More **Bandwidth**
- Higher packets-per-second (**PPS**)
- Consistent lower **latency**

### EBS Optimized

- **EBS** = Block storage over the **network**
- Historically network was **shared** .. **data** and **EBS**
- EBS Optimized means **dedicated capacity** for EBS
- Most instances **support** and have **enabled by default**
- Some support, but enabling costs extra

# Route 53 - Global DNS

### R53 Zone Concepts

- A **R53 Hosted Zone** is a DNS DB for a domain e.g. animals4life.org
- **Public** = Hosted on R53 provided **public DNS Servers**
- **Globally resilient** (multiple DNS Servers)
- Created with domain registration via R53 - can be created separately
- Host **DNS Records** (e.g. A, AAAA, MX, NS, TXT ...)
- Hosted Zones are what the DNS system references - **Authoritative** for a domain e.g. Animals4life.org

### Health Check Concepts

- Health check are **separate from**, but are **used by** records
- Health checkers located **globally**
- 10s or 30s
- TCP, HTTP/HTTPS, HTTP/HTTPS with String Matching
- **Healthy** or **Unhealthy**
- Endpoint, CloudWatch Alarm, Checks of Checks (Calculated)

### R53 Routing Policies

- Simple
- Failover
- Weighted
- Latency-based
- Geolocation
- Multi-value

# Relational Database Service (RDS)

### Relational (**SQL**) vs Non-Relational (**NoSQL**)

- Structured Query Language (**SQL**)
- Structure **in** & **between** tables of data - Rigid **Schema**
- ... Relationships between tables
- NoSQL - **Not one single thing --- different models**
- Generally a much more relaxed Schema
- Relationships handled differently

### Why might you do it ...

- Access to the DB Instance **OS**
- **Advanced DB Option tuning** ... (DBROOT)
- ... Vendor demands..
- **DB or DB Version AWS don't provide..**
- Specific **OS/DB Combination** AWS don't provide
- Arquitecture AWS don't provide (replication/resilience)
- Decision makers who '**just want it**'

### Why you shouldn't really..

- **Admin overhead** - managing EC2 and DBHost
- **Backup** / DR Management
- EC2 is **single AZ**
- **Features** - some of AWS DB products are amazing
- EC2 is **ON** or **OFF** - no serverless, no easy scaling
- **Replication** - skills, setup time, monitoring & effectiveness
- **Performance** .... AWS invest time into optimisation & features

### Relational Database Service RDS Architecture

- Database-as-a-service (**DBaaS**)
- .. **DatabaseServer**-as-a-service
- **Managed Database** Instance (1+ Databases)
- Multiple engines **MySQL**, **MariaDB**, **PostgreSQL**, **Oracle**, Microsoft **SQL Server**..
- .. **Amazon Aurora**

### RDS High-Availability (Multi AZ)

- **No Free-tier** - Extra cost for standby replica
- Standby **can't be directly used**
- **60-120** seconds failover
- **Same region only** (only AZs in the VPC)
- Backups taken **from Standby** (removes performance impact)
- AZ Outage, Primary Failure, Manual failover, Instance type change and software patching

### RPO: Recovery Point Objective

- Time between last backup and the incident
- Amount of maximum data loss
- Influences technical solution & cost
- Generally lower values cost more

### RTO: Recovery Time Objective

- Time between the DR event and full recovery
- Influenced by process, staff, tech and documentation
- Generally lower values cost more

### RDS Restores

- Creates a **NEW RDS Instance** - **new address**
- Snapshots = **single point in time**, creation time 
- Automated = **any 5 minute point in time**
- Backup is restored and transactino logs are 'replayed' to bring DB to desired point in time
- Restores **aren't fast** - Think about **RTO** Recovery Time Objective

### RDS Read-Replicas

- **5x** direct read-replicas per DB instance
- Each providing an **additional instance of read performance**
- Read-Replicas can have read-replicas - **but lag starts to be a problem**
- **Global** performance improvements

### Availability Improvements

- Snapshots & Backups Improve RPO
- **RTO's are a problem**
- RR's offer **nr. 0 RPO**
- RR's can be **promoted quickly** - **low RTO**
- **Failure only** - **until promoted**
- **Global availability improvements ... global resilience**

### Aurora Key Differences

- Aurora architecture is **VERY** different from RDS...
- ...Uses a "***Cluster***"
- A single **primary** instance + **0** or more **replicas**
- No local storage - uses **cluster volume**
- Faster provisioning & improved availability & performance

### Aurora Storage Architecture

- All SSD Based - **high IOPS**, **low latency**
- Storage is billed based on **what's used**
- **High water mark** - billed for the most used
- Storage which is freed up can be re-used
- Replicas can be added and removed without requiring storage provisioning

### Aurora Cost

- **No free-tier option**
- Aurora doesn't support Micro Instances
- Beyond RDS singleAZ (micro) Aurora offers better value
- Compute - hourly charge, per second, 10 minute minimum
- Storage - GB-Month consumed, IO cost per request
- 100% DB Size in backups are included

### Aurora Restore, Clone & Backtrack

- Backups in Aurora work in the same way as RDS
- Restores create a **new cluster**
- Backtrack can be used which allow **in-place rewinds** to a previous point in time
- Fast clones make a new database MUCH faster than copying all the data - **copy-on-write**

### Aurora Serverless Concepts

- Scalable - **ACU** - Aurora Capacity Units
- Aurora Serverless cluster has a **MIN & MAX ACU**
- Cluster adjusts based on load
- Can go to **0** and be **paused**
- Consumption billing per-second basis
- Same resilience as Aurora (6 copies across AZs)

### Aurora Serverless - Use Cases

- **Infrequently** used applications
- **New** applications
- **Variable** workloads
- **Unpredictable** workloads
- **Development** and **test** databases
- **Multi-tenant** applications

### Aurora Global Database

- **Cross-Region Data Recovery and Business Content**
- **Global Read Scaling - low latency performance improvements**
- **~1s or less** replication between regions
- **No impact** on DB performance
- Secondary regions can have **16 replicas**
- .. Can be promoted to R/W
- Currently MAX 5 secondary regions...

### Aurora Multi-Master

- Default Aurora mode is **Single-Master**
- **One R/W** and **0+ Read Only** Replicas
- Cluster Endpoint is used to write, read endpoint is used for load balanced reads
- Failover takes time - replica promoted to R/W
- In Multi-Master mode **all instances are R/W**

### Database Migration Service DMS

- A managed database migration service
- Runs using a **replication instance**
- **Source** and **Destination Endpoints** point at ...
- **Source** and **Target** Databases
- **One endpoint MUST be on AWS**

# Network Storage

### Elastic File System EFS

- **EFS** is an implementation of **NFSv4**
- EFS **Filesystems** can be **mounted in Linux**
- **Shared** between many EC2 Instances
- Private service, via **mount targets** inside a VPC
- Can be accessed from on-premises - **VPN** or **DX**

### Elastic File System EFS

- **Linux Only**
- **General Purpose** and **Max I/O** Performance Modes
- General Purpose = **default** for 99.9% of uses
- **Bursting** and **Provisioned** Throughput Modes
- **Standard** and **Infrequent Access** (IA) Classes
- Lifecycle Policies can be used with classes

# HA & Scaling

### Load Balancing Fundamentals

- Clients connect to the **Load Balancer**
- ... specifically the **listener** of the LB
- The LB connects on your behalf to 1+ targets (servers)
- 2 connections .. **listener** & **backened**
- Client **Abstracted** from individual servers
- Used for **High-Availability**, **Fault-Tolerance** and **Scaling**

### Application Load Balancer ALB

- ALB is a '**layer-7**' LB - **understands HTTP/S**
- Scalable and highly-available
- Internet-Facing or Internal
- **Listens** on the outside -> Sends to **Target(s) (Groups)
- **Hourly** rate and **LCU** Rate (Capacity)

### Exam Hints and Tips

- **Targets** => **Target Groups** which are addressed via **rules**
- Rules are **path based** or **host based**
- Support EC2, ECS, EKS, Lambda, HTTPS, HTTP/2 and Websockets
- ALB can use Server Name Indication **SNI** for **multiple SSL Certs** - **host based rules**
- Recommended vs Classic Load Balancer CLB (Legacy)

### Launch Configuration and Launch Templates

- Allow you to define the configuration of an EC2 instance **in advance**
- AMI, Instance Type, Storage & Key pair
- Networking and Security Groups
- Userdata & IAM Role
- Both are NOT editable - defined once. Launch Templates has versions.
- Launch Templates provide **newer feature** - including T2/t3 Unlimited, Placement Groups, Capacity Reservations, Elastic Graphics

### Auto Scaling Groups

- **Automatic Scaling** and **Self-Healing** for EC2
- Uses **Launch Templates** or **Configurations**
- Has a **Minimum**, **Desired** and **Maximum** Size (e.g 1:2:4)
- **Provision** or **Terminate** Instances to keep at the **Desired** level (between Min/Max)
- **Scaling Policies** automate based on metrics

### Scaling Policies

- **Manual** Scaling - Manually adjust the desired capacity
- **Scheduled** Scaling - Time based adjustment - e.g. Sales..
- **Dynamic** Scaling
  - **Simple** - "CPU above 50% +1", "CPU Below 50 -1"
  - **Stepped** Scaling - Bigger +/- based on difference
  - **Taget tracking** - Desired Aggregate CPU = 40% .. ASG handle it
- **Cooldown Periods** ...

### Final Points

- Autoscaling Groups are free
- Only the resources created are billed ...
- Use cool downs to avoid rapid scaling
- Think about **more**, **smaller** instances - **granularity**
- Use with Application Load Balancer (ALB's) for elasticity - **abstraction**
- ASG defines **WHEN** and **WHERE**, Launch Templates defines **WHAT**

### Network Load Balancer NLB

- NLB's are **Layer-4** .. only understand **TCP** and **UDP**
- **Can't understand HTTP/S** but are faster - **~100ms** vs **400ms** for application load balancers
- Rapid scaling - **millions of requests per second**
- 1 Interface **w/ static IP per AZ**, can use **Elastic IPs** (**whitelisting**)
- Can do **SSL Pass through** (see next lesson)
- Can load balance **non HTTP/S applications** - doesn't care about anything above TCP/UDP

### SSL Offload & Session Stickiness

- Bridging
- Pass-through
- Offload

# Serverless and Application Services

### Event-Driven Architecture

- **No constant running** or **waiting** for things
- **Producers** generate events when something happens
- .. clicks, errors, criteria met, uploads, actions
- Events are delivered to **consumers**
- .. **actions are taken** & the system returns to waiting
- Mature event-driven architecture **only consumes resources while handling events** (serverless .... More on this soon)

### Lambda Concepts

- Function-as-a-Service (**FaaS**)
- Event-driven **invocation** (execution)
- **Lambda function** = piece of code in one language
- Lambda functions use a **runtime** (e.g. Python 3.6)
- Runs in a **runtime environment**
- You are billed only for the duration a **function runs**...
- Key component of **serverless** architecture ...

### Key Considerations

- Currently - 15 minute execution limit
- New runtime environment every execution - **no persistence**
- **Execution Role** provides permissions ..
- Load data **from** other services (e.g. S3)
- Store data **to** other services (e.g. S3)
- (free tier) **1M free requests** per month and **400000 GB-seconds** of compute time per month

### CloudWatchEvents and EventBridge Key Concepts

- If **X** happens, or at **Y** time(s) ... do **Z**
- EventBridge is ....... CloudWatch Events v2
- A **default** Event bus for the account
- .. In CloudWatch Events this is the only bus (**implicit**)
- EventBridge can have additional event busses
- Rules match incoming events ...(or schedules)
- Route the events to **1+ Targets** .. e.g. Lambda

### API Gateway

- API Gateway is a **managed API Endpoint** Service
- Create, Publish, Monitor and Secure APIs ... **as a service**
- Billed based on Number of **API Calls**, **Data Transfer** and additional performance **features** such as caching
- Can be used directly for serverless architecture
- Or during a architecture **evolution** ...

### What is Serverless

- Serverless **isn't one single thing**
- You manage **few**, **if any** servers - low overhead
- Applications are a collection of small & specialised functions
- .... **Stateless** and **Ephemeral** environments - duration billing
- **Event-driven** .. consumption only when being used
- **FaaS** is used where possible for compute functionality
- **Managed services** are used where possible

### Simple Notification Service (SNS)

- **Public AWS Service** - network connectivity with Public Endpoint
- Coordinates the sending and delivery of **messages**
- Messages are <= **256KB** payloads
- **SNS Topics** are the base entity of SNS - **permissions** and **configuration**
- A **Publisher sends** messages to a **TOPIC**
- **TOPICS** have **Subscribers** which **receive** messages
- e.g. ... HTTP(s), Email(-JSON), SQS, Mobile Push, SMS Messages & Lambda
- SNS used across AWS for notifications - e.g. CloudWatch & CloudFormation

### Simple Notification Service (SNS)

- Delivery **Status** - (including HTTP, Lambda, SQS)
- Delivery **Retries** - Reliable Delivery
- **HA** and **Scalable** (Region)
- Server Side Encryption (**SSE**)
- Cross-Account via **TOPIC Policy**

### Step Functions: Some problems with Lambda

- Lambda is **FaaS**
- **15-minute max** execution time
- Can be chained together
- Gets messy at scale
- Runtime Environments are **stateless**

### Step Functions: State Machines

- Serverless workflow .. **START** -> **STATES** -> **END**
- States are **THINGS** which occur
- Maximum Duration **1 year**..
- **Standard** Workflow and **Express** Workflow
- Started via API Gateway, IOT Rules, EventBridge, Lambda .....
- Amazon States Language (**ASL**) - JSON Template
- **IAM Role** is used for permissions

### Step Functions: States

- SUCCEED & FAIL
- WAIT
- CHOICE
- PARALLEL
- MAP
- TASK (Lambda, Batch, DynamoDB, ECS, SNS, SQS, Glue, SageMaker, EMR, Step Functions)

### Simple Queue Service (SQS)

- Public, Fully Managed, Highly-Available Queues - **Standard** or **FIFO**
- Messages up to **256KB** in size - **link** to large data
- Received messages are **hidden** (**VisibilityTimeout**)
- .. then either reappear (retry) or are explicitly deleted
- **Dead-Letter queues** can be used for problem messages
- ASGs can scale and Lambdas invoke based on queue length

### Simple Queue Service (SQS)

- Standard = **at-least-once**, FIFO = **exactly-once**
- FIFO (Performance) **3000 messages per second** with batching, or up to **300 messages per second without**
- Billed based on 'requests'
- 1 request = 1-10 messages up to 64KB total
- **Short** (immediate) vs **Long** (**waitTimeSeconds**) Polling
- Encryption at rest (**KMS**) & in-transit
- Queue policy ...

### Kinesis Concepts

- Kinesis is a **scalable streaming** service
- Producers **send** data into a kinesis **stream**
- Streams can scale from low to near infinite data rates
- Public service & highly available by design
- Streams store a **24-hour** moving window of data
- Multiple consumers access data from that moving window

### SQS vs Kinesis

- SQS **1** production group, **1** consumption group
- SQS: **Decoupling** and **Asynchronous** communications
- **No persistence** of messages, **no window**
- Kinesis designed for **huge scale ingestion** ..
- ... and **multiple consumers** .. **rolling window**
- Data **ingestion**, **Analytics**, **Monitoring**, **App Clicks**

# Global Content Delivery And Optimization

### CloudFront

- CloudFront is a global object cache (**CDN**)
- Content is **cached** in locations **close to customers**
- **Lower latency** and **higher throughput**
- **Load** on the content server is **decreased**
- It can handle **static** and **dynamic** content

### CloudFront Terms

- **Origin** - The source location of your content
- **Distribution** - The '**configuration**' unit of CloudFront
- **Edge Location** - Local infraestructure which hosts a cache of your data
- **Regional Edge Cache** - Larger version of an edge location. Provides another layer of caching

### AWS Certificate Manager ACM

- HTTP - Simple and Insecure
- HTTP**S** - **SSL/TLS** Layer of Encryption added to HTTP
- Data is encrypted **in-transit**
- Certificates **prove identity**
- Signed by a **trusted authority**
- Create, renew and deploy certificates with ACM
- Supported AWS Services **ONLY** (e.g. CloudFront and ALBs..**NOT EC2**)

### Securing S3 using an Origin Access Identity (OAI)

- Origin Access Identity is **associate** to CloudFront Distribution
- Edge Locations via **OAI** Allowed
- Direct access **Blocked** via Implicit **DENY**
- Once **OAI** is associated with the distribution accesses are **FROM** the **OAI**

### Lambda@Edge

- You can run **lightweight** Lambda at **edge locations**
- **Adjust** data between the **Viewer** & **Origin**
- Currently supports **Node.js** and **Python**
- Run in the **AWS Public Space** (Not VPC)
- **Layers** are **not supported**
- **Different Limits** vs Normal Lambda Functions

### Lambda@Edge Use Cases

- A/B testing - **Viewer Request**
- Migration Between S3 Origins - **Origin Request**
- Different Objects Based on Device - **Origin Request**
- Content By Country - **Origin Request**

### Global Accelerator

- Moves the **AWS network closer** to customers
- Connections **enter at edge** .. using anycast IPs
- **Transit over AWS backbone** to **1+** locations
- Can be used for **NON HTTP/S** (**TCP**/**UDP**) - **Difference from CloudFront**

# Advanced VPC Networking

### VPC Flow Logs

- Capture **packet Metadata** .. **NOT** packet **contents**
- Applied to a VPC - All interfaces in that VPC
- Subnet - interfaces in that Subnet ..
- Interface directly
- VPC Flow Logs are NOT realtime
- Destination can be S3 or CloudWatch Logs
- ICMP=**1**, TCP=**6**, UDP=**17**
- To and from **169.254.169.254**, **169.254.169.123**, **DHCP**, Amazon **DNS** Server & **Amazon Windows license** not recorded

### Egress-Only Internet Gateway

- With IPv4 addresses are **private** or **public**
- **NAT** allows **private IPs** to access **public networks**
- ... **without allowing** externally initiated connections (**IN**)
- With **IPv6** all IPs are **public**
- Internet Gateway (IPv6) allows all IPs **IN** and **OUT**
- Egress-Only is **outbound-only** for **IPv6**
- Egress-Only Gateway is **HA by default** across **all AZs** in the region - **scales as required**

### VPC Endpoints (Gateway)

- Provide **private access** to **S3** and **DynamoDB**
- **Prefix List** added to **route table** => **Gateway Endpoint**
- Highly Available (**HA**) across all AZs in a region by default
- Endpoint policy is used to control what it can access
- Regional ... **can't access cross-region** services
- **Prevent Leaky Buckets** - S3 Buckets can be set to private only by allowing access ONLY from a gateway endpoint

### VPC Endpoints (Interface Endpoints)

- Provide **private access** to AWS Public Services
- .... anything **NOT S3** and **DynamoDB**
- Added to **specific subnets** - an **ENI** - **not HA**
- For HA .. add **one endpoint**, to **one subnet**, **per AZ** used in the VPC
- Network access controlled via **Security Groups**
- **Endpoint Policies** - restrict what can be done with the endpoint
- **TCP** and **IPv4** ONLY
- Uses **PrivateLink**

### VPC Endpoints (Interface Endpoints)

- Endpoint provides a NEW service endpoint DNS
- e.g. **vpce-123-xyz.sns.us-east-1.vpce.amazonaws.com**
- Endpoint **Regional DNS**
- Endpoint **Zonal DNS**
- Applications can optionally use these, or ...
- **PrivateDNS overrides** the **default DNS** for services

### VPC Peering

- Direct encrypted network link between **two VPCs**
- Works **same**/**cross-region** and **same**/**cross-account**
- (**optional**) **Public Hostnames** resolve to **private IPs**
- **Same region SG's** can reference **peer SGs**
- VPC Peering does **NOT** support **transitive peering**
- **Routing** Configuration is needed, **SGs** & **NACLs** can filter

# Hybrid Environments And Migration

### Borger Gateway Protocol (BGP) 101

- Autonomous System (**AS**) - Routers controlled by one entity ... a network in BGP
- **ASN** are unique and allocated by IANA (**0-65535**), **64512** - **65534** are private
- BGP Operates over **tcp/179** - it's **reliable**
- **Not automatic** - peering is **manually configured**
- BGP is a **path-vector** protocol it exchanges the **best path** to a **destination** between **peers** ... the path is called the **ASPATH**
- **iBGP** = Internal BGP - Routing **within** an AS
- **eBGP** = External BGP - Rounting **between** AS's

### AWS Site-to-Site VPN

- AWS Site-to-Site VPN is a hardware VPN solution which creates a highly available IPSEC VPN between an AWS VPN and external network such as on-premises traditional networks. VPNs are quick to setup vs direct connect, don't offer the same high performance, but do encrypt data in transit. This lesson details the architecture and key concepts you need to be aware of for the exam
- A logical connection between a VPC and on-premises network encrypted using IPSec, running over the **public internet**
- Full High Availability HA - if you design and implement it correctly
- Quick to provision ... **less than an hour**
- Virtual Private Gateway (**VGW**)
- Customer Gateway (**CGW**)
- VPN Connection between the **VGW** and **CGW**

### VPN Considerations

- Speed Limitations ~ **1.25Gbps**
- Latency Considerations - **inconsistent**, **public internet**
- Cost - AWS hourly cost, GB out cost, data cap (on premises)
- Speed of setup - **hours** .. all **software** configuration
- Can be used as a backup for Direct Connect (**DX**)
- Can be used with Direct Connect (**DX**)

### AWS Direct Connect (**DX**)

- A **1 Gbps** or **10 Gbps** Network Port **into AWS**
- .. at a **DX Location** (**1000-Base-LX** or **10GBASE-LR**)
- .. to your **Customer Router** (requires **VLANS**/**BGP**)
- .. or **Partner Router** (if **extending** to your location)
- Multiple Virtual Interfaces (**VIFS**) over one DX
- **Private** VIF (VPC) & **Public** VIF (Public Zone Services)

### (DX) Considerations

- Takes **MUCH** longer to provision vs **VPN**
- DX Port provisioning is quick ... the cross-connect takes longer
- .. extension to premises can take **weeks/months**
- Use **VPN first** .. then **replace with DX** (Or leave as **backup**)
- Faster .. 40Gbps with Aggregation
- Low consistent latency, doesn't use business bandwidth
- **NO ENCRYPTION**

### AWS Transit Gateway (**TGW**)

- The AWS Transit gateway is a network gateway which can be used to significantly simplify networking between VPC's, VPN and Direct Connect
- It can be used to peer VPCs in the same account, different account, same or different region and supports transitive routing between networks
- **Network Transit Hub** to connect VPCs to on premises networks
- Significantly **reduces** network complexity
- Single network object - HA and Scalable
- **Attachments** to other networks types
- **VPC**, **Site-to-Site VPN** & **Direct Connect Gateway**

### Transit Gateway Considerations

- Supports **transitive routing**
- Can be used to create global networks
- Share **between accounts** using **AWS Resource Access Manager**
- Peer with **different regions** ... same or cross account
- **Less complexity** vs **w**/**o** Transit Gateway

### Storage Gateway

- Hybrid Storage Virtual Appliance (**On-premise**)
- It's capable of running in 3 modes: FILE, TAPE and VOLUME (Stored or Cached)
- **Extension** of **File** & **Volume** Storage into AWS
- **Volume** storage **backups into** AWS
- **Tape** Backups **into** AWS
- **Migration** of existing infrastructure to AWS

### Storage Gateway

- Tape Gateway (**VTL**) Mode
  - Virtual tapes => S3 and Glacier 
- **File** Mode - SMB and NFS
  - File Storage backed by S3 Objects
- **Volume** Mode (Gateway **Cache**/**Stored**) - iSCSI
  - Block storage backed by S3 and EBS Snapshots

### Snowball / Snowball Edge / Snowmobile : Key Concepts

- Snowball, Snowball Edge and Snowmobile are three parts of the same product family designed to allow the physical transfer of data between business locations and AWS
- Move large amounts of data **IN** and **OUT** of AWS
- Physical storage .. **suitcase** or **truck**
- Ordered from AWS **Empty**, **Load up**, **Return**
- Ordered from AWS **with data**, **empty** & **Return**
- For the exam know which to use ..

### Snowball

- Ordered from AWS, Log a Job, Device Delivered (not instant)
- Data Encryption uses KMS
- **50TB** or **80TB** Capacity
- 1 Gbps (RJ45 1 GBase-TX) or 10 Gbps (LR/SR) Network
- **10 TB** to **10 PB** economical range (**multiple devices**)
- Multiple devices to **multiple premises** ..
- Only storage

### Snowball Edge

- Both **Storage** and **Compute**
- **Larger capacity** vs Snowball
- **10** Gbps (RJ45), **10/25** (SFP), **45/50/100** Gbps (QSFP+)
- **Storage Optimized** (**with EC2**) - 80TB, 24 vCPU, 32 Gib RAM, **1 TB SSD**
- **Compute Optimized** - 100TB + 7.68G NVME, 52 vCPU and 208 GiB RAM
- **Compute with GPU** - As Above .. **with a GPU**
- Ideal for remote sites or where data processing on ingestion is needed

### Snowmobile

- Portable DC within a shipping container on a **truck**
- Special order
- Ideal for single location when **10 PB+** is required
- Up to **100PB** per snowmobile
- Not economical for **multi-site** (**unless huge**) or **sub 10PB**

### Directory Service : What's a Directory?

- Stores **objects** (e.g. Users, Groups, Computers, Servers, File Shares) with a **structure** (domain/tree)
- Multiple trees can be grouped into a **forest**
- Commonly used in **Windows Environments**..
- Sign-in to multiple devices with the same username/password provides centralised management for assets
- ... Microsoft Active Directory Domain Services (**AD DS**)
- AD DS most popular, open-source alternatives (**SAMBA**)

### What about Directory Service?

- **AWS Managed** implementation
- Runs within a **VPC** ..
- To implement **HA** ... deploy into **multiple AZs**
- Some AWS services **NEED** a directory e.g. **Amazon Workspaces**
- Can be **isolated..**
- .. or **integrated** with existing **on-premises system**
- Or act as a '**proxy**' back to on-premises

### Picking between Modes..

- **Simple AD** - The default. Simple requirements. A directory in AWS.
- **Microsoft AD** - Applications in AWS which need **MS AD DS**, or you need to **TRUST AD DS**
- **AD Connector** - use AWS Services which need a directory **without storing any directory info in the cloud** ...  proxy to your on-premises Directory

### AWS DataSync

- AWS DataSync is a product which can orchestrate the movement of large scale data (amounts or files) from on-premises NAS/NAS into AWS or vice-versa
- Data Transfer service **TO** and **FROM** AWS
- **Migrations**, **Data Processing Transfers**, **Archival**/**Cost Effective Storage** or **DR**/**BC**
- .. designed to work at **huge scale**
- Keeps **metadata** (e.g. **permissions**/**timestamps**)
- Built in **data validation**

### AWS DataSync : Key Features

- **Scalable** - 10Gbps per agent (~100TB per day)
- **Bandwidth Limiters** (avoid link saturation)
- **Incremental** and **scheduled** transfer options
- **Compression** and **encryption**
- **Automatic recovery** from transit errors
- AWS **Service integration** - S3, EFS, FSx
- Pay as you use ... per GB cost for data moved

### DataSync Components

- **Task** - A 'job' within DataSync, defines what is being synced, how quickly, FROM where and TO where
- **Agent** - Software used to **read** or **write** to on-premises data stores using **NFS** or **SMB**
- **Location** - every task has two locations FROM and TO. E.g. Network File System (**NFS**), Server Message Block (**SMB**), Amazon **EFS**, Amazon **FSx** and Amazon **S3**

### FSx for Windows File Server

- Fully managed **native windows** file servers/shares
- Designed for **integration** with **windows environments**
- Integrates with **Directory Service** or **Self-Managed AD**
- **Single** or **Multi-AZ** within a VPC
- **On-demand** and **Scheduled** Backups
- Accessible using **VPC**, **Peering**, **VPN**, **Direct Connect**

### FSx Key Features and Benefits

- **VSS** - User-Driven Restores
- Native file system accessible over **SMB**
- **Windows permission model**
- Supports **DFS** .. scale-out file share structure
- Managed - no file server admin
- Integrates with **DS** AND **your own** directory

### FSx for Lustre

- Managed **Lustre** - Designed for **HPC** - **LINUX** Clients (**POSIX**)
- **Machine Learning**, **Big Data**, **Financial Modelling**
- **100's GB/s** throughput & **sub millisecond** latency
- Deployment types - **Pesistent** or **Scratch**
- Scratch - Highly optimised for **Short term** no replication & fast
- Persistent - **longer term**, **HA** (**in one AZ**), **self-healing**
- Accessible over **VPN** or **Direct Connect**

### FSx for Lustre

- Metadata stored on Metadata Targets (**MST**)
- Objects are stored on called object storage targets (**OSTs**) (**1.17TiB**)
- **Baseline** performance based on **size**
- Size - min **1.2TiB** then increments of **2.4TiB**
- For **Scratch** - Base **200 MB/s** per **TiB** of storage
- Persistent offers **50MB/s**, **100 MB/s** and **200 MB/s** per **TiB** of storage
- Burst up to **1,300 MB/s** per TiB (Credit System)

### FSx for Lustre

- Scratch is designed for **pure performance**
- **Short term** or **term** workloads
- **NO HA** .. **NO REPLICATION**
- **Larger file systems** means **more servers**, **more disks** and **more chance of failure !!**
- Persistent has **replication** within **ONE AZ** only
- **Auto-heals** when hardware failure occurs
- You can **backup to S3** with **both** !! (Manual or Automatic 0-35 day retention)

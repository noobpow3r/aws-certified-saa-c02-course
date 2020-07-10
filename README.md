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

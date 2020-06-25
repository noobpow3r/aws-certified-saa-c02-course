# AWS Certified SAA-C02 Course
Notes for Adrian Cantrill course https://learn.cantrill.io/

# Virtual Private Cloud (VPC) Basics

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

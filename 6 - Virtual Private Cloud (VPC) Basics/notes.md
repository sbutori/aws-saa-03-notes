# VPC Sizing and Structure

Step through the design choices around VPC design and IP planning. How to design an IP plan for a business; designing a VPC

One of the first things to decide on is IP range; the VPC CIDR.
- IMPORTANT: This planning is critical and is not easy to change later
- What size should VPC be? How many services need to fit, each service has an IP, each occupies space in VPC
- Are there any Networks we can't use, or need to interact with?
- Be mindful of IP ranges other VPC's used in other cloud env's in on-premise networks. Try to avoid IP ranges that other parties use that you might need to interact with
- Try to predict the future... How things may change
- VPC structure - Tiers (Web, Application, Database -- things that separate different application components and allow different security to be applied), Resiliency (Availability) Zones

### VPC Sizing / Structure - Example: Animals4Life (A4L).
*NOTE: REVIEW IP RANGES: Network address, prefix, how they map to range of address*
- 3 offices: London, NY, Seattle (3 IPs)
- Field workers (laptop, 3G, 4G, satellite, email, chat, file access, etc)
- Already has 3 existing networks. 192.168.10.0/24, 10.0.0.0/16, 172.31.0.0/16 (new AWS network design cannot use or overlap with these)
- Access/Migrate data. Avoid the following ranges:
  - 192.168.10.0/24 range is 192.168.10.0 -> 192.168.10.255
  - 10.0.0.0 (AWS) 10.0.0.0 -> 10.0.255.255
  - 172.31.0.0 (Azure) 172.31.0.0 -> 172.31.255.255 (this shares same IP address range as AWS defulat VPC, se we can't use default VPC for anthing production)
  - 192.168.15.0/24 (London) -> 192.168.15.255
  - 192.168.20.0/24 (NY) -> 192.168.20.255
  - 192.168.25.0/24 (Seattle) -> 192.168.15.255
  - 10.128.0.0/9 (google) -> 10.255.255.255
- When designing an IP addressing plan, don't use any of the above

### What IP ranges? How many networks does A4L need?
- A AWS VPC has a min range of /28 (16 IPs) and a max of /16 (65536 IPs)
- Avoid common ranges to avoid future issues. 10.1 and 10.0 are common, avoid up to 10.10. Start at 10.16 per Cantrill's recommendation
- How many ranges a business requires? First, how many regions does AWS operate in? Since we're pre-allocating, overestimate for a buffer. A4L we don't really know how many regions they'll operate in, but we can make educated guess then add buffer.
  - Suggest having at least 2 ranges in each region, in each AWS account
  - Assumed regions for A4L: 3 US, Europe, Australia (5) x2, and 4 AWS accounts. 3US + 1AU + 1EU = 5 Regions, 2 ranges ea. region, total of 10 IP ranges per account. Since there are 4 accounts, 10 ranges x 4 accounts = 40 IP ranges

#### IP Range Review:
- We're going avoid 10.0 - 10.10 (too commonly used)
- Start at 10.16 per Cantrill
- Can't use 10.128 -> 10.255 because Google Cloud uses it
- Our Range: 10.16 -> 10.127 inclusive

### VPC Sizing

| VPC Size    | Netmask | Subnet Size | Hosts/Subnet* | Subnets/VPC | Total IPs* |
| ----------- | ------- | ----------- | ------------- | ----------- | ---------- |
| Micro       | /24     | /27         | 27            | 8           | 216        |
| Small       | /21     | /24         | 251           | 8           | 2008       |
| Medium      | /19     | /22         | 1019          | 8           | 8152       |
| Large       | /18     | /21         | 2043          | 8           | 16344      |
| Extra Large | /16     | /20         | 4091          | 16          | 65456      |
- Deciding which to use on this chart? Two important Questions:
  - How many subnets will you need in each VPC?
  - How many IPs total? How many IPs per subnet?

#### VPC Sizing - How many Subnets?
- Services inside a VPC use subnets, which are where IP addresses are allocated from
- VPC Services run from within subnets
- A subnet is located in ONE availability zone

##### How many Availability Zones will your VPC use?

- Start with 3 as DEFAULT, per Cantrill. And add at least 1 spare. DEFAULT: AT MINIMUM, always assume 4 AZ's
- Within VPCs, you also have tiers. Tiers for different types of infrastructure inside VPC; web tier, application tier, database tier... so 3, plus buffer. DEFAULT: assume 4 tiers.
- 4 tiers, 4 AZ's. Each tier has a subnet in each AZ: 4x4 = 16 subnets. Using /16 VPC into 16 subnets, results in 1 smaller network ranges, each being /20. Or you could divide into subnets that are larger, but with fewer subnets per VPC.
TOTAL: 16 subnets with /20 prefix

### VPC Sizing / Structure - Example: Animals4Life (A4L) - CONTINUED - PROPOSAL:
- A4L is a global org and could grow significantly (assume huge level of growth)
- Use the 10.16 -> 10.127 range (avoiding Google, common ranges, etc)
- 5 AZs
  - Start at 10.16 (US1), 10.32 (US2), 10.48 (US3), 10.64 (EU), 10.80 (AU) - each account has 1/4th of the range
  - total of 16 /16 network ranges for each region.
- 3 accounts: general, prod, dev. Add 1 more for buffer. 4 accounts total
  - Each account in each region gets 4 /16 ranges, enough for 4 VPCs per region per account
  - /16 per VPC - 3AZ (+1), 3 tiers (+1) = 16 subnets for each VPC
  - /16 splits into 16 subnets = /20 per subnet (4,091 IPs) - May seems excessive, but assume highest possible growth potential

### VPC Sizing & Structure Resources
https://aws.amazon.com/answers/networking/aws-single-vpc-design/
https://cloud.google.com/vpc/docs/vpc
https://github.com/acantril/aws-sa-associate-saac02/tree/master/07-VPC-Basics/01_vpc_sizing_and_structure


## Custom VPCs, includes DEMO

Step through the architecture and features of Custom VPCs including the main issues which are raised in the exam
- Learn to build a complex, multi-tier VPC

In this multi-tier custom VPC build...
- Using VPC us-east-1, 10.16.0.0/16
- Inside VPC, 4 tiers in 4 AZs = 16 subnets.
- We will create all 4 tiers: reserve, DB, app, web
- We will only create 3 AZ's: A, B, C. We will not create subnets in the capacity reserved for the future AZ
- We will be creating VPC, subnets, an internet gateway, NAT gateways, bastion host (using bastion host is bad practice - don't do it, not secure), NACLs


## DEMO - Custom VPCs - Create VPC - Overview
- VPCs are **regionally isolated and regionally resilient service**. Operates from all AZs in that region.
- Nothing is allowed IN or OUT of a VPC without explicit permission; provides isolated blast radius; if you have a proble inside a VPC, the impact is limited to that VPC and/or anything connected
- Custom VPCs allow for simple or multi-tier; flexible configuration
- Custom VPCs also provide Hybrid Networking
- When you create VPC, you can pick DEFAULT (shared hardware) or DEDICATED (dedicated hardware) tenancy
   - If you pick DEFAULT tenancy, you can choose on a per resource basis later on when provisioning resources whether it's shared or dedicated hardware
   - If you pick DEDICATED tenancy at VPC level, it's locked in. Any resources created inside the VPC have to be on dedicated hardware (which carry a cost premium compared to Default)
  - Recommendation: unless you really know you require dedicated, pick default.
 - By default, VPC uses IPv4 private/public IPs.
  - The private CIDR block is main method of IP communication for VPC
  - Public IPs are used when you want make resources public (Internet or AWS Public Zone)
- VPC is allocated ONE mandatory Private IPv4 CIDR Block
   - Primary Block has two main restrictions: min /28 (16 IPs), max /16 (65,536 IPs)
  - Optionally, you can add secondary IPv4 CIDR blocks after creation
 - Optional: VPC can use IPv6 by assigning /56 IPv6 CIDR to VPC (this is gaining more and more adoption)
 -- IMPORTANT: You can't pick a block of IP ranges like IPv4. Your range is either allocated by AWS, or customer can use their OWN IPv6 range
 -- IPv6 doesn't have Public/Private concept, the range is all Publicly routable by default

 - VPCs also have DNS. provided by R53
   - VPC DNS Adress is `Base IP + 2`, so if VPC is `10.0.0.0`, then DNS IP is `10.0.0.2`

 - **EXAM:** Two critical options for how DNS functions in a VPC
    1. `enableDnsHostnames` setting - Determines whether instances in the VPC are assigned DNS hostnames. If set to true, instances in the VPC are assigned DNS hostnames that are resolvable via Amazon-provided DNS. If set to false, instances in the VPC are not assigned DNS hostnames.
    2. `enableDnsSupport` setting - Controls whether DNS resolution is supported for the VPC. If set to true, the Amazon-provided DNS server is enabled in the VPC, allowing resolution of public domains (e.g., google.com) and internal AWS DNS names (e.g., private IP to hostname mappings for instances). If set to false, the DNS resolution feature is disabled, and the VPC cannot resolve any DNS queries. If this is set to false, `enableDnsHostnames` cannot be set to true.


 ### DEMO - Custom VPCs - Create VPC - DNS

 ## DEMO pt.1 - Custom VPCs - Create/Configure VPC
 Steps through the architecture and features of Custom VPCs including the main issues which are raised in the exam

 ### STEPS pt.1
 1. Create the VPC: VPC > Your VPCs > Create VPC > select VPC Only > name tag "a4l-vpc1" > IPv4 CIDR "10.16.0.0/16" > IPv6 CIDR block "Amazon-provided IPv6 CIDR block", default us-east-1 > Tenancy set to Default > Create VPC
 2. VPC Settings: In VPC > Actions dropdown, "Edit VPC Settings" > DNS Settings check both Enable DNS Resolution and Enable DNS Hostnames < Save (for any resources in this custom VPC, if they have public IP addresses, they'll also get public DNS names)

## VPC Subnets

Subnets are what services run from inside VPCs, within a particular Availability Zone. They're how you add function, structure, and resilience to VPCs
*NOTE:* With AWS diagrams, color Blue is private and Green is for Public (green = Go = Public)
- Subnets in a VPC start off Private and must be configured to become Public
- Subnets are **AZ Resilient**. Created within on AZ and can never be changed. If AZ fails, subnet (and any contained services) fails
**EXAM: Subnet can NEVER be in multiple AZs. 1 subnet is in 1 AZ (belongs to).** An AZ can have 0 or more subnets.
- The IPv4 CIDR allocated to the the subnet is a subset of the VPC CIDR Block (has to be within VPC allocated range)
**EXAM: CIDR that a subnet uses cannot overlap with any other subnets in the VPC**
- A Subnet can optionally be configured for IPv6 and allocated a IPv6 CIDR.
  - For IPv6, the VPC is a fixed size of /56 and the subnet size is fixed to be a /64. Only one IPv6 CIDR block can be allocated to a subnet (meaning /64 subset of a /56 block from the VPC, with space for 256 /64 ranges that each subnet can use).
- By default, subnets in a VPC can communicate with other subnets in the same VPC.

- 5 IPs inside every VPC subnet are RESERVED
  - Example: Subnet is 10.16.16.0/20, range of 10.16.16.0 -> 10.16.31.255
  - Unusable Addresses: `Network` Address (10.16.16.0) - General, identifies the network.
  - Unusable Addresses: `Network+1` Address (10.16.16.1) - AWS Specific, VPC Router; logical network device that moves data between subnets
  - Unusable Addresses: `Network+2` Address (10.16.16.2) - AWS Specific, Reserved VPC address, but generally for DNS
  - Unusable Addresses: `Network+3` Address (10.16.16.3) - AWS Specific, Reserved Future Use
  - Unusable Addresses: Network `Broadcast` Address (10.16.31.255) - General. Last IP in subnet, normally used to reach all devices in network, even though VPC don't use it, it's still reserved.
  **EXAM: if a subnet has 16 usable IPs... it actually only has 11, because 5 are reserved (3 for AWS (+1, +2, +3, 1 network (1st IP in range), one broadcast (last IP in range))**

- *Dynamic Host Configuration Protocol (DHCP)*. It's how computing devices receive IP address automatically. VPC has a configuration object applied to it called DHCP Option Set, which is applied to a VPC at a time and flows through to subnets.
  - You can create DHCP option sets, but you CANNOT edit them. To change settings, you need to create a new DHCP and change the VPC allocation to the new DHCP Option Set.
  - On every subnet, you can define two IP Allocation Options:
    1. auto-assign public IPv4 address
    2. auto-assign an IPv6 address

 ### DEMO - Implement multi-tier VPC subnets

 1. Create Multiple Subnets with VPC multi-tier structure: VPC > Subnets > Create subnet > Select VPC (a4l-vpc1) > subnet name "sn-reserved-A" > IPv4 CIDR block "10.16.0.0/20" > IPv6 CIDR block, choose the one option, fill in unique IPv6 value with /64 CIDR block and the correct IP for the respective subnet (00 for first one, 01 for the second...) > Add New Subnet > Repeat for each subnet in the AZ (4 total) > Verify Details in each > Create Subnet. Repeat for AZB, AZC.
 2. Auto allocate IPv6 Addressing: select sn-app-A > Actions drop down, "Edit Subnet Settings" > Auto-assign IP settings, check "Enable auto-assign IPv6 address" > Save > Do this for all new subnets

 ### Custom VPCs - Resources
 - VPC Limits https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html
 - Architecture https://raw.githubusercontent.com/acantril/aws-sa-associate-saac03/main/0800-VIRTUAL_PRIVATE_CLOUD(VPC)/00_LEARNINGAIDS/VPCStucture-1.png
 - CIDR review: https://aws.amazon.com/what-is/cidr/
 - Subnet Calculator : https://www.site24x7.com/tools/ipv4-subnetcalculator.html


 ## VPC Routing, Internet Gateway, & Bastion Hosts
Step through the architecture and functionality of Route Tables, Routes, Associations, the internet gateway and public IP v4 functionality within a VPC

### VPC Routing

#### VPC Router

- Every VPC has a VPC Router - Highly Available
- In every subnet, router has the IP address of `network+1`.
- The VPC Router routes traffic between subnets.
- Controlled by `Route Tables` which influence what to do with traffic (each subnet has one)
- A VPC is created with what's known as a `Main route table` - this is the subnet default (if you don't explicity associate a route table with a subnet, it gets the `Main` route table of the VPC).
- A subnet can only have **ONE** route table associated with it at any one time. But a Route Table can be associated with many subnets

- A Route Table is just a list of routes. When traffic leaves the subnet the VPC router reviews the IP packets, looks at their destination address and identifies the routes that match that address in the Route Table. The VPC router then forwards that packet to its target destination, determined by the `Target` field on the route.
  - If multiple routes match, the prefix is used as a priority. The higher the prefix value, the more specific the route, the more priority a route has (e.g. a /30 will have priority over /16).
  - The `Target` field will either point at an AWS Gateway or it will say `local` (meaning the destination is in the VPC itself and may be forwarded just by the router)
  - **Routes with a `local` target ALWAYS have priority,**, no matter the CIDR range. They are always present and can't be updated.

- **EXAM:** Route tables are attached to 0 or more Subnets. A Subnet HAS to have a Route Table. It's either the `Main` route table of the VPC it belongs to or a custom route table. A route table controls what happens to data as it leaves the subnet(s). Local Routes are always present, un-editable, and match the VPC IPv4 or IPv6 CIDR range. For anything else, higher prefix values are more specific and have higher priority (except routes with a `local` target which always have highest priority, even if a more specific prefix exists (i.e. a local 10.0.0.0/16	route will have precedence over a 10.0.1.0/24 route created on its range)).

### Internet Gateway

A Internet Gateway (IGW) is used by the VPC in order to allow data to *exit-to* and *enter-from* the Public Internet or AWS Public Zone.

- **EXAM**: Internet Gateway (IGW) is a **REGION RESILIENT** gateway attached to a VPC.
  - A VPC can have no IGW's (Private VPC) or **ONLY 1 IGW**.
  - An IGW can be created and not attached to a VPC, but it can be attached to **ONE VPC** at a time.
  - If an IGW is attached to a VPC, it is available in all AZs that the VPC uses.

- IGW runs within the AWS Public Zone (more specifically, from the border of the Public Zone and VPC)
- IGW traffic between the VPC and the Internet/AWS Public Zone (S3,SQS,SNS and others)
- Managed; AWS handles the performance

#### Using an IGW
1. Create IGW
2. Attach IGW to VPC
3. Create a custom Route Table
4. Associate the Route Table with the subnet(s) inside the VPC
5. Add IPv4 (and optionally IPv6) default routes (0.0.0.0/0; ::/0) to the route table with target being IGW
6. Config subnets to allocate IPv4 addressess (and optionally IPv6 addresses by Default)

### How IPv4 Addressing works inside a VPC

- Example: IPv4 EC2 Instance <-> IGW <-> Linux Update Server
  - EC2 instance trying to send updates to Linux server.
  - The IPv4 instance has the private IP of 10.16.16.20, and IPv4 Public IP of 43.250.192.20. However, this isn't how it really works. what actually happens with public IPv4. They never touch the actual services in a VPC. When you allocate a public IPv4 address, for example to this EC2 instance, a record is created which the IGW maintains, linking the instances Private IP to it's allocated Public IP.
  - A packet for transit has Source and Destination IP addresses. At this point, packet is not configured with any public addressing (not currently routable accross public internet, could not currently reach the Linux update server). Packet leaves instance, follows Default route to IGW. IGW sees packet is from EC2 based on source IP, and IGW knows the associated IPv4 address for this Source IP and changes Source address to IPv4 to be routable across internet. Linux Update server receives packet from Source IPv4 address. On the way back, the inverse happens.

- **EXAM:** At no point is the O/S on the EC2 instance aware of the public IP, it just knows of its private IP (no IGW translation)
  - NOTE: Since IPv6 is publicly routable, the O/S on the EC2 instance would know it and it gets passed through IGW to sent without changing to a different IP (unlike IPv4 in the example above)

### Bastion Hosts (Jumpbox)
Bastion Hosts = Jumpbox. Entry point for private-only VPCs.

- Instance in a public subnet
- Architecturally incoming managements connections arrive there
  - Once connected, can then go on to access internal-only VPC resources
- Generally the only entry point to a highly secure VPC


## DEMO - Configuring A4L public subnets and Jumpbox

Implement an Internet Gateway, Route Tables and Routes within the Animals4life VPC to support the WEB public subnets. Once the WEB subnets are public, we create a bastion host with public IPv4 addressing and connect to it to test.

* Jumpboxes are bad - but you need to understand how to spot bad things *

Currently in our VPC, all subnets are private and can't be used for communication with the internet or AWS Public Zone. We're going to reconfig the three "web" subnets to be public.

1. Create IGW: VPC > Internet Gateways > Create internet gateway > name "a4l-vpc1-igw" > Create
2. Attach IGW to VPC: In a4l-vpc1-igw, Actions dropdown "Attach to VPC" > select your VPC > Attach
3a. Make subnets Public, create Route Table (two Routes per subnet, IPv6 and IPv4): VPC > Route Tables, Create route table > Name "a4l-vpc1-rt-web" > select a4l VPC > Create
3b. Make subnets Public, config subnets: Route Table > select a4l rt > Subnet associations tab > Edit subnet associations > select web subnets (A,B,C) > Save > Routes tab
4. Add additional routes to Route Table (default routes for IPv4/6): VPC > Route Tables > select a4l rt > Routes tab, Edit Routes > Add Route > destination for IPv4 0.0.0.0/0 > target: internet gateway, select yours > Add another route, destination "::/0" (targets all IPv6), target: IGW > Save changes
5. Allocate resources with IPv4 public addresses: Subnets > select web A > Actions > Edit Subnet Settings > check "Enable auto-assign public IPv4 address" > Save> Repeat for web B and C
6. Test configuration: EC2 > Launch Instance > name "A4L-BASTION" > Quickstart: Amazon Linux 2023 > Architecture: 64 bit, instance type: t2 micro, keypair: a4l > Netowkr Settings, VPC: A4L-VPC1, Subnet "sn-web-a", auto assign IPv4/6 "Enable", select "creat esecurity group" and name/description "A4L-BASTION-SG" > Launch Instance
7. Connect to Instance: Right-click instance, "Connect" > EC2 Connect > Connect > SSH tab, copy ssh command > Open Terminal, CD to Downloads, paste command to connect > Yes for fingerprint thing
8. Cleanup: Select A4L instance, Instance State: Terminate > VPC, select A4L vpc, Actions > Delete VPC

- NOTE: When connecting to EC2 instances that have IPv4 address, you can use EC2 instance connect or Local SSH client. Also, Session Manager can connect you if you don't have IPv4 addressing.


## Stateful vs Stateless Firewalls

TCP and IP Review: They work together.

- Transmission Control Protocol. Connection based protocol. When you make a TCP connection, each side is sending IP packets to each other (they have a source/destination IP). TCP is a Layer 4 Protocol that lies on top of IP. It adds error correction with the idea of ports. HTTP is TCP Port 80, HTTPS is TCP port 443.

**EXAM:** Every CONNECTION between user/server has two parts: 1. REQUEST (initiation) 2. RESPONSE.

What actually happens:
1. Client pick temporary "ephemeral" port
2. Client initiates server connection REQUEST using well-known port number (like Port 443 HTTPS)
3. Server RESPONDS using source port of tcp/443 and ephemeral port picked by client

- Directionality depends on your perspective. What is INBOUND/OUTBOUND -- Source/Dest swap depending on who's perspective.

### Stateless Firewalls

Ex. Server is within a stateless firewall; meaning that it doesn't understand the state of connections (it sees REQUEST/RESPONSE as separate parts, so you need 2 rules for each connection).

- TWO Rules needed for each connection, 1 IN; 1 OUT
- Because the firewall is stateless, it doesn't know which specific port is used for the response. You'll often have to allow the FULL RANGE of ephemeral ports to any destination (1024–65535 per RFC 6056)
  - This makes security engineers uneasy, which is why Stateful Firewalls are preferred.

### Stateful Firewalls

A Stateful Firewall is intelligent enough to ID the REQUEST and RESPONSE components of a connection as being related.

- ONE RULE per connection; Allowing the REQUEST (inbound or outbound) means the RESPONSE (IB or OB) is automatically allowed; this reduced admin overhead and the chance for mistakes
- You DO NOT need to allow full Ephemeral Port range

## Network Access Control Lists (NACLs)

A network access control list (NACL) is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets. You might set up network NACLs with rules similar to your security groups in order to add an additional layer of security to your VPC.

- NACLs are STATELESS (the request and response are viewed as different things; there's no link between the two)
- NACL only impacts data crossing the subnet boundary (communications between instances in the SAME subnet are not affected)
- Can EXPLICITLY ALLOW AND EXPLICITLY DENY; you can block specific IPs/IP Ranges associated with bad actors
  - Rules are processed in order, lowest rule number first. Once a match occurs, processing STOPS. '*' is an implicit DENY, if nothing else matches
- NACLS not aware of Logical Resources. They only allow you to use IP and CIDR ranges, ports and protocols
- NACLs cannot be assigned to AWS resources, ONLY subnets
- Often used together with Security Groups to add an implicit DENY (Use SGs to ALLOW traffic, NACL to DENY)
- Subnet can only have *ONE* NACL (Default or Custom, see below), but a NACL can be associated with *many* subnets

- Each network ACL contains TWO sets of rules. Inbound and Outbound rules/protocols. Rules match the destination (DST) IP/Range or the DST Port, together with the protocol and ALLOW or DENY based on that match
  - NACL response to client requires using the ephemeral port range (chosen at random by client/requester)

### NACLs and multi-tiered architecture (operating across multiple subnets)

Becomes more complex with multi-tiers. Internal communications also required IB/OB rules on the app subnet NACL. If you have a CLIENT (external), a WEB subnet and APP subnet, connection rules are required for all IB/OB movements whether in between web/app subnets or between client and web subnet

- For every single communication, the rule-pairs IB/OB are needed on each NACL for each communication type which occurs:
  1. Within a VPC
  2. TO a VPC
  3. FROM a VPC

- Default NACL: A VPC is created with a Default NACL that has one set of IB/OB rules with implicit deny (\*) and a catch-all IB/OB set of ALLOW ALL rule (0.0.0.0/0). The result is that *all traffic is allowed*, the NACL has no effect.

- Custom NACL: Created for a specific VPC and initially NOT associated with subnets. Only ONE IB/OB rule by default: implicit deny (\*) -- so at first, if you associate subnets with a custom NACL, *all traffic is denied*.

- You can create multiple custom NACLs within a single VPC. NACLs are strictly tied to the VPC in which they are created, and they only apply to subnets within that VPC.

## VPC Security Groups (SG)

Security Groups (SGs) are another security feature of AWS VPC, only unlike NACLs they are attached to **AWS resources**, not VPC subnets. SGs offer a few advantages vs. NACLs in that they can recognize AWS resources and filter based on them; and they can reference other SGs and also themselves. But SGs are not capable of explicitly blocking traffic - so you often require assistance from NACLs.

- SGs are STATEFUL; if you allow IB/OB request, the response is automatically allowed
- Major limitation of SGs: **NO EXPLICIT DENY**. Can be used to allow traffic or NOT Allow traffic (which is an implicit Deny). This means they can't be used to block specific bad actors
-- This is why you use NACLs in conjunction with SGs, because NACLs are used to add the explicit denies
- Supports IP and CIDR, but also allows referencing AWS Logical Resources, including other SGs and ITSELF within rules
- SGs are NOT attached to Instances/Subnets, they are attached to *Elastic Network Interfaces (ENI)*. An instance has a primary ENI.

**EXAM:**
- Security groups (SGs) are attached to **ELASTIC NETWORK INTERFACES (AWS Resources)**, not instances/subnets
- SGs **cannot EXPLICITLY DENY traffic**, they must be paired with NACL to have an Explicit Deny

### SG - Logical References

Ex. Within VPC, we have APP and WEB subnets. Both APP and WEB are protected by SGs. External User is accessing web app via port tcp/443, allowing 0.0.0.0/0. Web to App is using tcp/1337. How do you best allow communication between WEB and APP? To take advantage of extra SG features, we could reference the web security group within the APP SG. In this, the Source for the IB Rule on APP is specifically the logical resource reference of the WEB SG

- **EXAM:** An SG Reference applies to anything which has the SG attached. When you're referencing an SG from another SG, you're actually referencing any resources which have the *SG associated with them.*

### SG - Self Referencing

Ex. Private Subnet in AWS with ever-changing number of app instances. An IB Rule in each instance can have its own SG referenced, allowing traffic between any other instances with that SG.

**EXAM:** SG Self referencing handles IP changes automatically.
**EXAM:** Also allows for simplified management of any intra-app communications (Eg. Microsoft Domain Controls, or apps with high availability within clusters.

NOTE: A logical resource is an abstraction that doesn't cost you anything.  It's a definition.  Something becomes a physical resource when it starts to cost you in terms of compute and store. In terms of cloud formation, you can define logical resources that are then used to create physical resources.  Within the template, you refer to things using the logical name.  Once things are deployed, you'll see deployed resources having physical IDs. In essence, a physical resource in AWS is a realized resource that consumes AWS infrastructure and services rather than a logical representation of one.


## NAT (Network Address Translation) and NAT Gateway

What is NAT? A process of giving a private resource outgoing-only access to the internet. Set of processes which adjusts IP packets by changing their source or destination IP addresses from the private IPs to the Public NAT Gateway IP.

- IGW performs a type of NAT known as "Static NAT"; it's how a resource can be allocated with a public IPv4 address (1:1 IP Translation)
- NAT, on the other hand, is many private IPs to one single IP (Many:1 IP Translation), this is popular as IPv4 addresses are running out.
  - *IP Masquerading:* hides a whole private CIDR Block behind ONE PUBLIC IP
  - NAT gives this private CIDR range **OUTGOING** access to the public internet and the AWS Public Zone

- AWS can provide NAT services in TWO ways:
1. EC2 instance config'd with NAT (cheaper, historical)
2. NAT Gateway - Configured inside VPC (better, managed)

- NAT is not required for IPv6, only IPv4. All IPv6 addresses are PUBLICLY ROUTABLE
- Need bi-directional connectivity on an instance? use IPv6 default route and point to IGW as target: ::/0 Route+IGW
- Outgoing ONLY access for an IPv6 instance? You need ::/0 Route + "Egress-Only IGW"

### NAT Gateway (we'll use the term 'IP Masquerading' interchangeably)

NAT Gateway (NATGW) is provisioned into a PUBLIC subnet (like WEB subnet, in our examples), which can use public IP addresses. The Private Subnet has a route table that can point all its instances to the NAT Gateway within the WEB subnet.

- NATGW contains a Translation Table, where all required source packet data is stored: IP address, src, dst, port #, etc
- A NATGW modifies (translates) the private source IP address of the packet from a subnet to the NATGW's private IP address. It then sends the packet to the Internet Gateway, which assings a public IP address to the NATGW. The package is then sent to and back from the destination.
- The NATGW job is to allow multiple private IPs to share (masquerade as) a single public IP address. This is also how your internet router works.

- NATGWs have to be run from a PUBLIC subnet, so remember you need:
  - an IGW,
  - IPv4 allocation enabled on subnets,
  - and default routes for the subnets pointing at the IGW
- Elastic IPs - NAT Gateways use a special type of public IP address called Elastic IPs. These are IPv4 addresses that are static and don't change.
- NATGW's are an **AZ resilient** service, since they are provisioned into a public subnet inside an AZ.
  - To attain Region Resilience like with an IGW, you need:
    - one NATGW in EACH AZ you're using in a VPC; and
    - a Route Table for the private subnets in that AZ pointing to the NATGW in a public subnet also in that AZ.

**EXAM:**
- NATGW's are a managed service. Scales to 45 Gbps. If you need to scale more, you can use multiple NATGWs in different subnets in the same AZ.
- NATGW's Billing:
  - Hourly charge for running it (per hour, and partial hours are billed as full hours)
  - \+ data processing charge (per GB of processed data)
- FALSE - Exam may suggest 1 NATGW is sufficient, that 1 is regionally resilient. FALSE. If you want regional resilience, you have to deploy NATGW into each AZ you use.
- NATGW's are AVAILABILITY ZONE RESILIENT. If you want regional resilience, you have to deploy NATGW into each AZ you use.
- NAT/NATGW isn't required and **DON'T work with IPv6**

### Scenarios where a NAT instance may be preferred over EC2+NATGW:
- NAT used to be provided by NAT EC2 instances.
- If you value availability, bandwidth, low maintenance, and high performance, choose NATGW
- But if you want to allow an EC2 instance anyway to function as a NAT instance, Disable `Source/Destination Checks`. Otherwise, the EC2 Network Interface by default will drop all traffic that is not either the source or destination of the EC2 instance.
- NAT Instance will fail if anything in the EC2 instance fails. A NATGW is highly available in 1 AZ--highly available (automatically recover and scale), but will still fail entirely if AZ fails entirely. For MAX Availability, have a NATGW in every AZ you use.
- If Cost is the primary issue, a NAT Instance can be cheaper. Especially at very low or very high volumes.

**EXAM:**
- NAT Instance can be used as a Bastion Server
- NAT Instances are just EC2 instances (can use SGs)
- NATGW CANNOT be used as Bastion Server, cannot do port-forwarding, b/c you can't connect to its operating sysytem (AWS Managed)
- NATGW does NOT support Security Groups; you can only use NACLs with NATGW
- NATGW is NOT Free Tier eligible


## DEMO - NAT Gateways - Implementing private internet access using NAT Gateways

Implement a highly-available regionally resilient NAT Gateway solution within the Animals4life VPC. You will create three Nat Gateways, Route tables and Default Routes with the Nat Gateway as a target and finally associate those Route Tables with the Reserved, App and DB subnets in AZ A, B and C before testing the solution using the internal instance.

1. One-click deployment: click lesson link > acknowledge
2. Confirm resources done provisioning: CFN > Stacks Resources > A4L Resources tab > Click Instance ID for A4LINTERNATEST > On EC2, Connect to instance with Session Manager -- You'll see no internet connectoin
NOTE: Session Mgr allows connection to an EC2 instance that has no public internet connectivity
3. Connect to Internet with NATGW: VPC > NATGQ, Create NATGW > name "a4l-vpc1-natgw-A", subnet sn-web-A, Elastic IP: click 'Allocate' > Create NATGW > Repeat for B, C
4. Configure Routing, Route Table: VPC > Route Tables > Create route table > select VPC a4l-vpc1, name "a4l-vpc1-rt-privateA" > Create > repeat for B,C
5. Create Default route in each RT: select RT "privateA" > Routes tab > Edit routes > Add route > DST 0.0.0.0/0, target NATGW A > Save Changes > Repeat for B, C
6. Associate Route Table with subnets: VPC Route Tables, select web-A, Edit subnet associations > select private subnets (app, db, reserved) > Save associations > Repeat for B, C
7. Cleanup: Edit subnet associations > uncheck all > save (x3) > Delete RTs > NATGWs: actions: delete (x3) > Elastic IPs: Release (x3) > CFN: Delete A4L stack

# AWS Public VS Private Services

Architecture of both Public/Private Services. "Private"/"Public" refer to networking only. Private runs within VPC, Public is S3
- Both require Permissions / Configuration
3 Zones:
- 1. Public Internet Zone
- 2. AWS Private Zone: VPC's are Virtual Private Clouds. VPCs are isolated unless configured otherwise; can attach IGW (internet gateway) for access to internet
- 3. AWS Public Zone (runs between Public and Private zones). It's a network connected to the Public internet

#### AWS Public Service: What is true of an AWS Public Service? 1. Located in the AWS Public Zone 2. Anyone can connect, but permissions are required to access the service
#### AWS Private Service: What is true of an AWS Private Service? 1. Located in a VPC 2. Accessibly from the VPC it is located in 3. Accessible from other VPCs or on-premise networks as long as private networking is configured


### AWS Global Infrastructure
AWS have created a global public cloud platform which consists of isolated 'regions' connected together by high speed global networking.

Two types of Deployment at global level: 1) AWS Regions, 2) AWS Edge Locations (doesn't directly map to continent or country)
#### AWS Region: Geographically spread. Full compute, storage, DB, AI, Analytics
-- Three main benefits
--- 1. Geographically separate for physical reslience. Isolated Fault Domain
--- 2. Geopolitical Separation. Different governance
--- 3. Location Control. Performance
-- Inside regions: You can use Region Code or Region Name. Eg Sygney Australia. RC: ap-southeast-2, RN: Asia Pacific (Sydney)

#### AWS Availability Zone (AZ)
AZ's are located inside regions. Eg. for Sydney AU: ap-southeast-2a, ap-southeast-2b, ap-southeast-2c. If an issue is only in 1 availability zone (AZ), you'll still have functioning services in the other AZs

#### AWS Edge Location: Local distribution points closest to user. Content distribution services, edge computing. Eg. Netflix storing content close as possible to user. Edge locations for fast/efficient data transfer because they're closer to the user. Uses CloudFront

#### AWS AZ VS Edge Location
An Availability Zone is an isolated location within an AWS region. An edge location delivers cached content to the closest location to reduce latency for users.

NOTE: Visit: infrastructure.aws

Service Resilience
- Globally Resilient: Operates globally with a single DB and replicated across multiple regions. World would need to fail to experience a full outage here. Eg. IAM
- Regionally Resilient: Operate in single region with one dataset in a region. Replicate data to AZs for resilience.
- Availability Zone Resilient: Prone to failure as they have less redundancies.

### AWS Default Virtual Private Cloud (VPC)
A Virtual Network inside AWS. VPCs are regional services; regionally resilient. They operate from multiple AZ's in a region.
- A VPC is 1 account and 1 region, cann't be spread across multiple accounts/regions
- Private and isolated unless configured otherwise
- Default VPC (max 1 per region, pre-configured) and Custom VPCs (can have many)
- Can be used to connect AWS private networks to on-premise network. Or connecting during multi-cloud deployments

EXAM: VPCs are REGIONALLY resilient

#### Default VPC
There can only be one default VPC per region, and they can be deleted and recreated from the console UI. Unless configured otherwise, VPC is entirely private/isolated.
- VPC CIDR: Start and end range of IP addresses VPC can use. If anything needs (and is allowed to) communicate with VPC, it needs to communicate to the VPC CIDR
- Default VPC only gets 1 CIDR IP range. Can't change it.
- Default VPC provides Internet Gateway (IGW), Security Group, and NACL
- VPC can be subdivided into subnetworks. Each subnet in VPC is located in one AZ. Default VPC has one subnet in each AZ by default
-- Each subnet uses part of the VPC CIDR range
-- Default VPC CIDR: always 172.31.0.0/16
-- Assigns public IPv4 addresses
--- /20 subnet in each AZ in the region. The higher the /#, the smaller the network. /17 is half the size of /16

QUIZ: How many subnets are in a default VPC? Default VPCs always have the same IP range and same '1 subnet per AZ' architecture. # Subnets = # AZ's in region

EXAM: Default VPC CIDR IP? 172.31.0.0/16

##### Default VPC - Create / Delete

To Locate: AWS Dashboard > Search "VPC" > Your VPCs

To Delete: check Default VPC > actions dropdown "Delete" > follow prompt

To Create Default VPC: If you've deleted Default VPC. In Your VPCs > Actions dropdown > Create Default VPC

### Elastic Compute Cloud (EC2) Basics
Basically the default compute service of AWS. It allows you to provision Virtual Machines known as Instances with resources you select and an operating system of your choosing.
- EC2 is IaaS, Infrastructure-as-a-service. Unit of consumption is the Instance. Instance is a configured O/S
- Private service by default, uses VPC networking
- Public access must be configured. The VPC it's running within must support Public access (if using custom VPC, you need to configure this in VPC)
- EC2 is AZ Resilient
- When launching an Instance, you can choose size and capability
- On-Demand Billing, by second or hour
- Storage Types: Local on-host storage, or network storage via Elastic Block Store (EBS)
- By default, AWS has a [soft] limit of 20 instances per region

EXAM: EC2 is AVAILABILITY ZONE resilient

#### EC2 instance has attribute called State
States to Know: Running, Stopped, Terminated
- When instance is launched and provisioned, it moves to Running. If instance is shut down, it is Stopped
- Instance can be terminated, but this cannot be reversed

These States influence the charges for an instance.
- Stopped instance uses less resources, therefore less expensive. You won't be charged for "running" the instance
- Regardless of Running or Stopped, storage is still allocated and you'll still be charged for EBS storage even if instance is stopped
- To guarantee 0 cost on instance, you must Terminate. Termination is irreversible

#### Components for instance charge: running the instance, storage the instance uses, extras for any commercial software the instance is launched with

#### AMI: Amazon Machine Image
Image of an EC2 instance
- Can create EC2 instance
- Can be created FROM an EC2 instance

##### AMI contains attached permissions. Controls which accts can/can't use AMI:
- Public Type - Everyone allowed
- Owner - Implicit Allow
- Explicit - Allow specific AWS accounts

##### AMI includes the following:
- One or more Amazon Elastic Block Store (Amazon EBS) snapshots, or, for instance-store-backed AMIs, a template for the root volume of the instance (for example, an operating system, an application server, and applications).
- Launch permissions that control which AWS accounts can use the AMI to launch instances.
- A block device mapping that specifies the volumes to attach to the instance when it's launched.

##### NOT stored in AMI:
- Instance Settings
- Network Settings

EXAM: What is NOT stored in an AMI? Instance and Network Settings

Root Volume: AMI contains boot volume of instance, boots O/S

Block Device Mapping: Config that links volumes that AMI has and how they're presented to O/S. Ie. Which is boot volume, which is data volume

#### Connecting to EC2:
- Connect to Windows instances using RDP (Remote Desktop Protocol), port 3389
- Connect to Linux distros use SSH protocol, port 22. Log in using SSH key pair

### DEMO: Create an EC2 Instance
https://learn.cantrill.io/courses/1820301/lectures/41301621
1. Create SSH Key Pair: Navigate to EC2 Console (search bar) > left sidebar > Network & Security > Key Pairs) > click "Create Key Pair"
- Note: For creating key pair, Private key file format choises are .pem and .ppk. For MacOS/Linux, always use .pem, if using modern Windows you can choose .pem. Older Windows or Putty terminal app, choose .ppk
2. Configure: Left sidebar "Instances" > click "Launch Instances" > select O/S (Amazon Linux) > select keypair login (A4L) > Network Settings: select Create Security Group > name/description "MyFirstInstanceSG" > leave Inbound security groups rules as defaults > Launch EC2 Instance

### DEMO: Connect to Terminal of an EC2 Instance
1. In EC2 Dashboard, Right Click your EC2 instance, select "Connect" > SSH Client
2. Open your local Terminal / Command Prompt and change directories to wherever your SSH private file key is (likely Downloads, it's a .pem file created previously)
3. Copy the command at the bottom of the SSH Client tab "ssh -i "A4L.pem" ec2-user@ec2-54-89-175-122.compute-1.amazonaws.com", verify fingerprint with "yes"
- Note: If using MacOS/Linux and "Permission Denied": paste in terminal "chmod 400 A4L.pem"
- More key file permissions help: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html#connection-prereqs-private-key



### S3 Basics: Simple Storage Service
Global Storage Platform - regional based/resilient. Access from anywhere.
- Public service, can cope with unlimited data & multi-users
- Perfect for hosting large amounts of data
- Economical and accessed via UI/CLI/API/HTTP

#### S3 Delivers 2 things: Objects and Buckets

##### S3 Objects
What is an Object? An object is a file and any optional metadata that describes the file.

Two components and some associated metadata:
1. Object Key. identifies object in a bucket
2. Value. data or contents of the object. Object size can range from 0 bytes to 5 terabytes (TB) in size.
Metadata: Version ID, Metadata, access control, subresources

NOTE: S3 = Default Storage Service in AWS

##### S3 Buckets (Simple Storage Service)
Containers created in a specific AWS region; default place to go to in order to configure S3
- Data inside a bucket has a primary home region. Never leaves this region unless configured to do so
- Blast radius of a failure is limited to a region
- Buckets can hold an unlimited number of objects; infinitely scalable
- S3 bucket has no complex structure; flat-structure; everything stored at same level; all stored at Root level (if you list out on Command, it'll look like there are folders, but there aren't Eg. /old/photo1.jpg). Folders are called 'prefixes' in S3.


# EXAM: Bucket name must be Globally Unique. Error where you can't create a bucket? Prolly not a unique name

# EXAM: More bucket name restrictions: 3-63 characters, all lowercase, no underscores, not formatted like IP addresses

# EXAM: Bucket limits: soft limit of 100 buckets per AWS account, hard limit of 1000 buckets (hard limit increased by connecting with Support). If you have more than 1000 users, you can't use 1 bucket per user, but you can use prefixes within a bucket to let multiple users use one bucket.

# EXAM: Unlimited Objects in a bucket, ranging from 0 bytes to 5TB in size

# EXAM: Object Structure: Key = Name, Value = Data

#### S3 Patterns and Anti-Patterns
- S3 is an Object Store system, NOT File System and NOT Block System
- S3 has no File System, it's flat. It's not Block storage, so you can't mount it as K:\ or /images
- Great for 'offload'
- S3 should be your default INPUT and/or OUTPUT to MANY AWS products

# EXAM: Where to store data in AWS? S3 should be default answer

# QUIZ: What is true of Simple Storage Service (S3)
- S3 is an AWS Public Service
- S3 is an Object Storage System
- Buckets can store an unlimited amount of data

### DEMO: Creating an S3 Bucket
Create, interact with bucket, upload to bucket, interact with uploaded objects
- As S3 is Global, you can't select a Region in advance

STEPS: Move to S3 Console > click 'Create bucket' > name 'koalacampaign987654123' (must be globally unique) > select Region (default N.VA) > untick "Block *all* Public Access (this just enables you to Enable public access later) > click "Create Bucket"

#### Amazon Resource Name (ARN): All resources in AWS have a unique identifier, the ARN.
To see bucket ARN, Amazon S3 > Buckets > click the bucket you want > 'Properties' tab > Amazon Resource Name

#### DEMO: Upload files to Bucket
Amazon S3 > Buckets > click the bucket you want > Objects, click Upload > Select Files > Use default settings > click Upload
- Now, you'll see these files in the Object tab

Note on creating a folder: it doesn't actually create a folder, it creates an object that appears to be a folder. They are just emulated using prefixes Eg. archive/koalazzz.jpg -> Not actually a file, but named to look like it
- Without versioning enabled, same name files can overwrite each other. With Versioning, they can't

#### DEMO: Delete Buckets in S3
Two step process:
1. Empty the bucket > select Bucket, click Empty
2. Re-select Bucket, click Delete

### CloudFormation (CFN) Basics
A tool which lets you create, update, delete infrastructure in AWS using Templates
- At its base, CFN uses templates. Can create/update/delete templates
- Template written in YAML or JSON

From the Documentation: AWS CloudFormation is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you

#### What makes a template? Components:
- All templates have a list of resources. This is the only mandatory item
- Description. Free text field for the author to provide details on template. (If you have AWSTemplateFormatVersion, Description MUST follow it in YAML/JSON
- Metadata. Can control the UI (groupings, order, labels, descriptions), as well as other things (to be covered later)
- Parameters. Value parameters, default values, etc.
- Mappings. Create lookup tables.
- Conditions. Allows decision making in the template. Step 1 create condition, Step 2 use condition
- Outputs. Once template is finished it can present outputs like admin or setup adddress, instance ID

#### How does CFN use Templates?
Eg. Template that creates EC2 resource. Resources defined in CFN Templates are called Logical Resources.
- When template is given to CFN, CFN creates a stack which is a living and active representation of a Template. Stack is created with CFN does something with a template.
- For any logical resources in the stack, CFN makes a corresponding physical resourse in your AWS account. It's CFNs job to keep logical and physical resources in sync. Physical resource: A resource created by creating a CloudFormation Stack
- If you delete a CFN Stack, its logical resources are deleted and CFN subsequently deletes the physical resources

# EXAM: In a CFN Template, if you have the AWSTemplateFormatVersion item, the Description MUST follow directly after in your YAML/JSON template file

### DEMO: Simple Automation w/ CFN; Creating EC2 with CFN

"CloudFormation" or CFN in Search > click "Create Stack" > Template is ready > Upload a Template File > use Demo file (ec2instance.yaml) > click "Next" > Name: "cnfdemo1" > leave the rest default > "Next" > skip config "Next" > Review cfndemo1: Capabilities, check the box
- Template auto-creates an s3 bucket when you upload a Template
- YAML file:
-- LatestAmiID: Instead of giving latest AMI ID, you can make Type a "latest version" AMI
-- CloudFormation Functions are used within YAML Outputs to get values like InstanceID, Availability Zone (AZ), Public IP of EC2, etc. Eg. !Ref, !GetAtt
-- Resources Component is the main one in the file. In this temmplate, an instance role and instance role profile are being created (SessionManagerRole, SessionManagerInstanceProfile) -- covered later.
-- Also being created in the Resources portion of the template: EC2 instance, InstanceSecurityGroup. Port 22 (SSH)/Port 80 (HTTP)
-- Resources: EC2Instance: Properties: InstanceType: "t2.micro" -- Free tier elligible
- After Submitting stack to be created, you'll see that each Resource being created is done so by an Event, "CREATE_IN_PROGRESS", "CREATE_COMPLETE". This whole process takes some time before all of cnfdemo1 is showing CREATE_COMPLETE

#### EC2 Session Manager: EC2 Dashboard > Connect to Instance > Session Manager
- No need to SSH in now
- Two logical resources in YAML template configure this ability


## CloudWatch Basics
CloudWatch is a core supporting service within AWS which provides metric, log and event management services. Collects / manages operational data on your behalf.

### Three jobs:
1. Metrics. AWS Products, Apps, on-premises. Some metrics require extra installed CloudWatch Agent
2. CloudWatch Logs. AWS Products, Apps, on-premises. Almost anything logged can be ingested by Logs
3. CloudWatch Events. AWS Services & Schedules. If an AWS service does something, this will generate an event that can perform another action. Or you can set up a chrono-repeating event through here.

### Core Concepts
- Namespace: Container for monitoring data. There is a naming ruleset for naming. All AWS data foes into a special namespace AWS/[service]
- Metric: Collection of related data points in a time-ordered structure
- Datapoints: A single unit of a metric; consists of timestamp and a value
- Dimensions: Generally, a metric is a collection (Eg. CPU utilization comes from all EC2 instances, not just one). You can use dimensions to single out resources to see their individual metrics. "Separate datapoints for different things or perspectives within the same metric"
- Alarms: Taking actions based on metrics. States: OK or ALARM (ALARM can be SNS Notification or Action), and INSUFFICIENT DATA (when there's not yet enough data)


## DEMO: Monitoring with CloudWatch
https://learn.cantrill.io/courses/1820301/lectures/41301629
1. Create EC2 Instance: Nav to EC2 service > Launch Instance Defaults for all, but you can pay to Enable Detailed CloudWatch Monitoring
2. Create CloudWatch alarm: Nav to CloudWatch dashboard > All Alarms > Create Alarm
3. Select Metric > EC2 > Browse Tab (make sure you know last 4 of Instance ID number) > Last 4 of ID number, check box for CPUUTilization metric
4. Metrics / Contitions: Conditions: Static, Whenever CPU Util is Greater/Equal 15% > "Next" > Remove Notification > Alarm Name "cloudwatchtestHIGHCPU" > Next/Create Alarm
5. Back to EC2, connect to EC2 with Instance Connect
6. Need to install Stress app > command: sudo yum install stress -y
7. Stress Test: command: stress -c 1 -t 3600. Make sure CloudWatch alarm is OK status before entering this command.
8. Clean up: Delete CloudWatch Alarm and Terminate EC2 instance. Delete EC2 Security Group

Detailed CloudWatch Monitoring: Enables you to monitor, collect, and analyze metrics about your instances through Amazon CloudWatch. Additional charges apply if enabled. If no value is specified the value of the source template will still be used. If the template value is not specified then the default API value will be used.


## Shared Responsibility Model
Vendor and User shared responsibilities. This applies from a security perspective so you know which elements you and the vendor manage.
- Customer is responsible for security 'in' the cloud: client-side data encryption, server-side encryption, networking traffic protection, O/S, platform, IAM, apps
- AWS responsible for security 'of' the cloud


## High-Availability (HA) vs Fault-Tolerance (FT) vs Disaster Recovery (DR)
### High-Availablily - Aims to ensure an agreed level of operational performance, usually uptime, for a higher than normal period. Doesn't aim to stop failure, but online and providing services as often as possible. Fix should be as quick as possible, often with automation to bring systems back into service quickly; maximizing a system's online time.
- System availability is in the form of percentage uptime. Three 9's -- Up 99.9% of the time=8.77 hours/year downtime. Five 9's=5.26 minutes/year down time
- Costs required to implement: design decisions and automation
### Fault Tolerance - The property that enables a system to continue operating properly in the event of the failure of some (one or more faults within) of its components. Fault tolerance systems work through failure with no disruption; operating through failure.
- Cost of fault tolerance can be expensive because of redundancies and failure toleration via duplicate system components
- Example: Plane operates with Fault Tolerance (more engines, electronic systems etc); can't tolerate failure in flight
### Disaster Recovery - Post-disaster recovery plan. A set of policies, tools, and procedures to enable recovery or continuation of vital technology infrastructure and systems following a natural or human-induced disaster. When disaster occurs, you want to avoiding losing irreplaceable items.


## Route53 (R53) Fundamentals
AWS's managed DNS product. Global service, single database; no need to pick region in console UI. Globally resilient.

### R53: Two main services:
1. Register Domains
2. Host Zones... managed nameservers

### R53: Architecture
Register domains. To do this, it has relationships with all major domain registries (.com, .io, .net, etc)
1. Check if domain available 2. Zone File created for registering domain (DNS DB for domain) 3. Allocates managed nameservers for this zone (usually 4). get nameserver records added to top level domain zone

### R53: Zones
DNS Zones and Hosting for those zones. Hosted zone created if domain available (and allocates 4 NS's to host zone). HZ can be Public or Private (linked to VPCs, for sensitive DNS records). Hosted Zones stores records (recordsets).

## DEMO: R53 Register Domain - animals4life.org
Search / Nav to R53 console > Registered Domains > click "Register Domains" > Search for domain > Select domain you want > Proceed to Checkout (choose duration, auto-renew) > fill out Contact Info > Pricing (up front cost + monthly hosted fee) > click "Submit" > await domain registry
- Transfer Lock: Security feature, domain can't be transferred away from R53 without disabling this lock
- If you delete and recreate HZ, you'll be allocated 4 new Name Servers (would need to update some stuff if you weren't using R5#)


## DNS Record Types
- Nameserver Records (NS). Allows delegation to occur in DNS. Eg. .com zone
- A Records / AAAA Records. Map host names to IP addresses. A record = IPv4, AAAA = IPv6
- CNAME Record. Canonical name. Let's you create equivalent of DNS shortcuts; host-to-host records. CNAMEs reduce admin overhead.
- MX Record. For email. How a server can find the mail server (SMTP) for a specific domain.
- TXT Records. Add arbitrary text to a domain; add additional functionality. Eg. Prove domain ownership.

EXAM: CNAMEs cannot point directly at an IP address, only other names.

QUIZ: What is a CloudFormation Logical Resource? A resource defined in a CFN Template
QUIZ: How many DNS Root Servers Exist? 13
QUIZ: Who manages DNS Root Servers? 12 Large Organizations
QUIZ: Who manages DNS Root Zone? IANA
QUIZ: A Record = IPv4, AAAA Record = IPv6
QUIZ: DNS Record type is how the root zone delegates control of .org to the .org registry
QUIZ: Which type of organization maintains the zones for a TLD (e.g .ORG)? Registry
QUIZ: Which type of organization has relationships with the .org TLD zone manager allowing domain registration? Registrar
QUIZ: How many subnets in a default VPC? Equal to the number of Availability Zones in the region the VPC in located in

### TTL - Time To Live
Numeric value that can be set on DNS records. Getting result from authoritative source is an authoritative answer. If a 3600 TTL value is set, results of query are stored in resolver server for 3600 seconds (1 hour), creating a non-authoritative answer for delivery -- during this time, another query would receive this cached non-authoritative answer.
- Low value means more queries against your server. High value less queries but less control
- TIP: If doing any work to change DNS values, it's recommended that you lower the TTL value in advance (days/weeks in advance)


## IAM Identity Policies
Policies attached to identities in AWS.
1. Understand their architecture 2. Gain ability to read/understand a policy 3. Learn to read/write your own

IAM Policy is a set of security statements to AWS, granting or denying product/features.
- IAM Identity Policy Document: 1 or more statements, created using JSON
-- Statement Id or Sid: Let's you identify a statement and what it does
-- Action: Can be specific individual action, a wild card, or list of multiple independent actions
-- Resources: Wild cards, lists of individual resources (using ARN name)
-- Effect: Allow or Deny

NOTE: Action and Resource within IAM Policy must match for statement to execute.

### How to Handle Overlap Type Situations
Priorities:
1. Explicit DENY: Overrules everything.
2. Explicit ALLOW: Allows access to all actions on any S3 resource (overrules all but Explicit DENY).
3. Default DENY (Implicit): No explicit DENY or ALLOW, so this takes effect. IAM ID's are generally restricted access by default.
- DENY (explicit), ALLOW (explicit), DENY (implicit)

### Three types of Policies
- 1. Inline Policies - JSON policies applied to each IAM account individually. (Manually applying policies to each Identity)
- Managed Policies - Created as their own object. You can attach this policy to more than one identity. Best for Common Access Rights
-- 2. AWS managed or 3. Customer managed
-- Reusable
-- Low Management Overhead
When to use Inline Policies? For Special or exceptional Allows or Deny's


## IAM Users and ARNs (Amazon Resource Name)
### IAM Users: An identity used for anything requiring long-term AWS access eg. Humans, apps, service accounts. If you can picture one thing; named thing, 99% of the time use IAM
- Starts with the principal; an entity trying to access AWS account (ppl, svc, group of  these_
- Authenticate principal. Principal needs to prove identity against IAM (using Username/PW (humans usually) or Access Keys (app or human in CLI))
- Principal becomes Authenticated Identity
- Authorization for actions via IAM Policy (statements)

### ARN (Amazon Resource Name):
Uniquely identify resources within any AWS accounts. ARNs are used in IAM policies and have a defined format

### ARN Format
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id

Note: s3 is globally unique, so no need to specify region...
arn:aws:s3:::catgifs -> references s3 bucket ITSELF
arn:aws:s3:::catgifs/* -> references objects IN the bucket (thse two ARNs do not overlap)

# EXAM: MAX 5,000 IAM Users per account
# EXAM: IAM User can be a member of 10 groups maximum
System with more than 5,000 identities? Can't use IAM user for each identity. IAM Roles & Identity Federation fixes this (more later)


## DEMO - Simple Identity Permissions Policies in AWS
Assign some permissions to an IAM identity
- 1-click deployment https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0052-aws-mixed-iam-simplepermissions/demo_cfn.yaml&stackName=IAM
-- Create IAM user Sally and two s3 buckets

IAM PW - Min. 8 characters. Uppercase, lowercase, numbers

Steps:
1. open first link, create PW, create stack
2. Login to new IAM Sally with incognito window, change PW, check ec2 and s3 that the permissions are limited
3. Using the lesson's attached .zip, open s3_fulladmin.json and copy the code to clipboard
4. In ADMIN account > IAM dashboard > User: Sally > Permissions > "Add permissions" > Create inline policy > JSON tab, paste copied text > name "s3admininline" and click "Create policy"
5. In s3, you can prove permissions by uploading merlin/thor.jpg through Sally IAM account

### DEMO - Delete all CFN / IAM info above
1. Delete Managed Policy: IAM > Users > Sally > Permissions > remove "AllsAllS3ExceptCats"
2. s3 > select "catpics"/"animalpics" buckets > click "Empty" > type "permanently delete", submit
3. CFN > Stacks > Delete Stack. NOTE: If you don't see this stack, check your region to make sure youre in N.Virginia


## IAM Groups
IAM groups are containers for IAM users. Makes managing IAM users easier.
- Groups can have both Inline and Managed policies attached
- Soft limit: 300 Groups per account (you can get Support to increase this)
- Groups are NOT a True Identity; they can't be referenced as a PRINCIPAL in a policy (you can grant access to users, not groups)
- Can hold Identity Permissions

EXAM: You can't login to IAM Groups; no credentials/login of their own
EXAM: IAM User can be a member of up to 10 IAM Groups (hard limit)
EXAM: AWS Account has a 5,000 IAM user limit (hard limit)
- There is no effective limit for the number of Users in a single IAM Group
EXAM: There is no "All Members" group by default. In IAM, you could create and add all Users to a group
EXAM: Groups CANNOT have nesting. No Groups within Groups
EXAM: Groups are NOT a True Identity; they can't be referenced as a PRINCIPAL in a policy (you can grant access to users, not groups)

Eg. If Sally is a part of 2 IAM Groups, each with an attached policy, and she has an attached policy, she has 3 policies which become Merged Policies
- For figuring out overlapping, combined all policies and tally Allows/Deny's and undergo the same DENY-ALLOW-DENY pattern

### Resource Policy: Controls access to a specific resource, allowing/denying identities to access that bucket. Does this by referencing this ID's with ARNs


## DEMO - Permissions control using IAM Groups
How groups can be used to hold permissions for group members

1. After prep steps complete, remove catpic permissions from Sally IAM > Users > Sally > Permissions > check cat Policy, "Remove"
2. IAM > User Groups > click "Create group" > name "developers" > select policy name "IAMGROUPS-AllowAllS3ExceptCats-7OLPCURC1UQF"
3. IAM > User Groups > tab Users > Add Users > select Sally > Add Users

### IAM Groups DEMO: Tidy Up:
1. IAM > User Groups > Developers > Remove AllowAllS3ExceptCats policy
2. IAM > Users Groups > delete Developers
3. Empy s3 Buckets . s3 > Buckets > click and Empty cat/animals buckets
4. CloudFormation > Stacks > delete IAMGROUPS


## IAM Roles - The Tech
- Principal: Physical person, app, device, or process which wants to Authenticate with AWS.
- IAM User: If you can think of a single entity as a principal, then prolly use IAM User
- IAM Roles: If you don't know the number of principals. Or more than 5,000 principals.
- A Role doesn't represent 'you', it represents a level of access inside an AWS account. Permissions borrowed for a short period of time
- Roles can be referenced within Resource Policies.

### IAM Roles Policies - Two Types
1. Trust Policy - The trust policy specifies which trusted account members are allowed to assume the role.
2. Permissions Policy - The permissions policy grants the user of the role the needed permissions to carry out the intended tasks on the resource.
- Temporary Security Credentials: Given when a Role gets assumed by something that is allowed to assume it. These are like access keys, but time limited.
-- Generated by STS, Secure Token Service. "sts:AssumeRole"
-- Checked against Permissions Policy


## IAM Roles: When to use IAM Roles
Examples for selecting when and when to not use IAM Roles.

- Example 1. Common use of Roles are for AWS Services themselves
-- Eg. AWS Lambda. Lambda function has no permissions by default, so it needs permissions to do things when it runs. Running Lambda Function is called "function invocation"/"function execution". We need to give this function permissions with access keys and there's a better way than hardcoding keys into script. To provide these permissions, we create an IAM Role called "Lambda Execution Role", which has a Trust Policy/Permissions Policy
-- This is good for a role because you'd have to hardcode into the function otherwise (security risk/update issues). Roles provide TSC (Temp security credentials)
-- For a given Lambda Function, you could have 0 to unknown number of copies running. Therefore, it makes sense to be a Role.

- Example 2. Emergency or out-of-the-usual situations
-- Eg. Group Helpdesk team given Read-Only access to a customer's AWS account to watch performance. Anything past read-only handled by a Senior team. However, if a customer calls at a time when Senior team is unavailable, the Helpdesk team may need more than read-only; Break Glass Situation. "Break glass for excepted access".
-- An emergency role can be assumed when Helpdesk user needs to "Break glass for more than read-only access"

- Example 3. Adding AWS to existing corporate environment
-- Eg. You have an existing physical network with an existing identity provider, Microsoft Active Directory. External Accounts / Identities cannot be used to directly access AWS Resources. For a corporate with more than 5,000 staff, you can't assign each employee an IAM (MAX 5000). Roles can re-use IAM Users
-- Allow IAM Role to be assumed by one of the External Entities (the identities in Microsoft Active Directory)
--- When Role is assumed, Temporary Security Credentials are used

TERM: ID Federation: Giving permissions to an external ID provider and allowing externals to assume a Role
TERM: Web Identity Federation. Eg. When you are given the option to login with Google, Facebook, etc (these are Web Identities), then use these to assume an IAM Role for AWS access.
-- Advantages: No AWS credentials on the external entity. Makes use of existing accounts. Scales to hundreds of millions of users and beyond.

EXAM: How you can architect solutions that will work for mobile apps: Use ID Federation
EXAM: External Accounts / External Identities cannot be used to directly access AWS Resources. They must assume a Role to gain access.

### IAM Roles: When to use Roles: Cross-Account Access
Eg. Two AWS accounts. Your Account and Partner Account
- Your account offers an application which processes scientific data and they want you to store data in Partner Account's s3 bucket. Your account has 1000s of ID's and they don't want this data in their account.
- In this situation, use a Role in the Partner Account. Any objects uploaded by this Role into the Partner Account means the Partner Account owns these objects
- Roles can be used cross-account to access individual resources, or access to whole accounts


## Service-linked Roles (SLR) & PassRole (PR)
A service-linked role is a unique type of IAM role that is linked directly to an AWS service. Service-linked roles are predefined by the service and include all the permissions that the service requires to call other AWS services on your behalf. The linked service also defines how you create, modify, and delete a service-linked role. A service might automatically create or delete the role. It might allow you to create, modify, or delete the role as part of a wizard or process in the service. Or it might require that you use IAM to create or delete the role.

### SLR: Key Difference between SLR and normal IAM Roles
- You can't delete a service linked role until it's no longer required / used within that AWS service

### SLR: Permissions needed to create - Policy
Key elements: top statement is Allow, action iam:CreateServiceLinkedRole, resource is NOT guessable--see link below
- https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html

### SLR: Role Separation
One group given ability to create roles, other group gets ability to use the SL Role

### PR: PassRole Permissions
To use pre-existing role with a service without having to create / edit a new role, you need to provide PassRole Permissions.

To pass a role (and its permissions) to an AWS service, a user must have permissions to pass the role to the service. This helps administrators ensure that only approved users can configure a service with a role that grants permissions. To allow a user to pass a role to an AWS service, you must grant the PassRole permission to the user's IAM user, role, or group.
- "Pass the role permissions"
- A method inside AWS that gives you the ability to implement Role Separation
-- An important AWS security architecture


## AWS Organizations
A product that allows larger businesses to manage multiple AWS accounts in a cost-effective way with little to no management overhead

Large orgs need to manage many AWS accounts (100s or more). Without AWS Orgz, these accounts would each have their own pool of IAM users and separate payment methods. Beyond 5-10 accounts, this becomes unwieldy.

### AWS Org - How it's created
First, take a single / standard AWS account (not within an org) and with this create an AWS Organization. The Org isn't created IN the account, you're just using the account to create the AWS Org. This standard AWS account that created the Org then becomes the Management Account for the org (previously Master Account).

Using this management account, you can invite other standard AWS Accounts into the org. As they are existing accounts, they need to approve the Org invites. If they approve, they become part of the Org. Once these Standard Accounts join the Org, they become Member Accounts

### AWS Org - Hierarchy of Containers (which contain Members and other containers)
Structure in AWS Orgs is hierarchical (upside-down tree). Top of hierarchy is Root Container of Organization (not to be confused with Account Root User)--this is a container for AWS Accounts, Mgmt and Member Accounts. This Org Root container also can contain other containers, known as Organizational Units (OU's): can contain AWS accounts or other Organizational Units

#### Types of Accounts relating to AWS Orgs
- Management Account / Master Account. Account that created the AWS Organization and invites Account Members
- Standard Account. A standard AWS account that is not part of an AWS Org
- Member Account. A standard AWS account that joined an AWS Org
NOTE: Only ONE Management Account per AWS Org

### AWS Org - Management Account
- Only ONE Management Account per AWS Org
- Account that creates the Org
- Account that handles Consolidated Billing

#### AWS Org - Consolidated Billing
Consolidated Billing. Eg. AWS Org with 4 accounts: 1 Mgmt and 3 Members that would each have their own billing information. However, since they're in an AWS Org, the individual billing methods are REMOVED for member accounts. Instead, billing is passed through to Management Account of the AWS Org (AKA Payer Account)
- Payer Account - Management Account that contains the payment method for an AWS Organization
- This helps remove Financial Management Overhead
- Bulk discounts; consolidation of reservations and volume discounts

#### AWS Org - Service Control Policies (SCP)
Allows you to restrict what Member Accounts can do.

#### AWS Org - New Account within Org
For new account, you just need a valid, unique email address. This way, there is no invite process (like there is with having pre-existing accounts join an AWS Org).

### AWS Org Changes Best Practice for User Logins / Permissions
With Orgs, you don't need IAM users inside every AWS Account. Instead, use Roles to allow IAM Users to access other accounts.
- Best practice is single account to handle all logins
- Large enterprise may use Identity Federation to pull in on-premise identities for logging in to login account
- Role Switch: Once Member accounts are logged in, they can Role Switch from login account into other Member Accounts

## DEMO - AWS Organizations
Steps:
1. The GENERAL account will become the MANAGEMENT / MASTER account for the organisation
2. We will invite the PRODUCTION account as a MEMBER account and create the DEVELOPMENT account as a MEMBER account.
3. Finally - we will create an OrganizationAccountAccessRole in the production account, and use this role to switch between accounts.

1. Nav to AWS Organization > "Create an Organization"
2. In Incognito Window, log in to the Production Account admin, copy Account ID > back to General AWS Management Account window > "Add an AWS account" > Invite Existing Account > Paste in ID / add message > Send Invitation
3. In Production Admin account > nav to AWS Organizations > Invitations > Accept Invitation
4. Manually adding a Role to an invited account > In Prod account nav to IAM > Roles > Create Role > Type of Entity: AWS Account > get account ID of General AWS Org Account > select Another AWS Account > Paste Account ID > Add Permissions: AdministratorAccess > Next > name "OrganizationAccountAccessRole" > Create Role. In IAM > Roles >  OrganizationAccountAccessRole > Trust Relationships, you'll see the General Account ID referenced as a Trusted Entity
5. Role Switch: From General to Production Account. Copy Production ID > Main Dropdown "Switch Role" > click "Switch Role" > paste Prod ID > name of Role created "OrganizationAccountAccessRole" > Display Name "Prod" > select color Red > Create / Switch Role. You'll see Display name "Prod" on top right menu drop down. You can also SWITCH BACK

6. Create new Account within Org. AWS Orgz > Add Account > Create an AWS Account > name "OrganizationAccountAccessRole" > Create
7. After new Account created, copy its account ID > dropdown Switch Role > switch to DEV role


## Service Control Policies (SCPs)
- A feature of AWS Organizations which allow restrictions to be placed on MEMBER accounts in the form of permissions boundaries. Establish which permissions can be granted in an account.
- SCPs can be applied to the organization, to OU's or to individual accounts
- Member accounts can be effected, the MANAGEMENT account cannot. But, it can indirectly attach to Account Root User
- SCPs DON'T GIVE permission - they just control what an account CAN and CANNOT grant via identity policies
- SCP is a JSON document
- SCPs inherit down the Org tree. Eg. If attached to Root Container, all Org accounts affected. If attached to OU, it affects the OU and all units / members within

TIP: SCPs DON'T GRANT any permissions, they just establish boundaries / limit permissions

### SCP: Allow / Deny Lists
- Allow List. Block by Default and allow certain services
- Deny List. Allow be Default and block certain services (Deny List is Default list, and default policy attached is FullAWSAccess)

To implement Allow Lists, 1. Remove AWS FullAccess Policy, 2. Add Allowed services into a new SCP

Best practice is Deny List for less admin overhead.

### SCP: Identity Policies in Accounts VS SCPs
EXAM: There is an overlap of permissions between these two types of policies. The OVERLAPPED permissions are what are actually active


## DEMO - Using Service Control Policies (SCP)
Update the structure within the organization, add hierarchy (Root, Dev OU, Prod OU) - and apply an SCP to the PRODUCTION account to test their capabilities
- Follows the same DENY-ALLOW-DENY IDP pattern

1. Create OU: Be in Admin account > AWS Organizations > check Root container box, "Actions: Create New" > Org Unit Name "PROD" > Create OU.
2. Create OU: Repeat above for DEV
3. Move relevant accounts to new OUs: check Production account, dropdown "Move" > Select respective OU (PROD/DEV) > Move AWS Account
4. Add SCP to Production Account: Switch Role: PROD > nav to s3 > Create Bucket > name (must be globally unique) "catpics3453451" > region: us-e-1 > upload attached photo (proving we have admin access)
5. Restrict with SCP. Switch Back > AWS Orgz > Policies > click SCP, enable SCP > Create Policy > copy .json text content from lesson file denys3.json > replace JSON with copied text > name "Allow All Except S3" > create policy
6. AWS Orgz > AWS Accounts > click PROD OU > tab "Policies" > Attach Allow All Except S3 Policy
7. Detach FullAWSAccess. AWS Orgz > Account PROD > tab "Policies" > select "attached directly" FullAWSAccess > Detach > Detach Policy
8. We reverted everything back to normal (re-attached Full Access policy to PROD OU, emptied s3 bucket, deleted s3 bucket


## Cloudwatch Logs
- CloudWatch Logs is a public service which can accept logging data, store it and monitor it
- It is often the default place where AWS Services can output their logging
- CloudWatch Logs is a public service and can also be utilised in an on-premises environment and even from other public cloud platforms
- Built-in AWS Service integrations: EC2, VPC Flow Logs, Lambda, CloudTrail, R53, and more
- Unified Cloudwatch Agent: How anything outside AWS can log data to Cloudwatch
- Metric Filter: Can generate metrics based on logs

### Cloudwatch - What it looks like
First, it's a Regional Service. Starting point: Logging Sources; external compute, DB's, external APIs, AWS services. Log Events are created from data from logging sources. Log Events are stored inside Log Stream (an ordered sequence of log events from the same source). Then a Metric Filter takes Log Group/Stream data, distills metrics which can bet set with alarm notifications

EXAM: CloudWatch is a REGIONALLY resilient service

### Cloudwatch - Vocab
- Log Group: Container for multiple log streams for the same type of logging. Log Group also stored config settings (retention, permissions)
- Log Stream: An ordered sequence of log events from the same source. Log Streams inherit Log Group config settings.
- Log Events: Created from data from logging sources
- Logging Sources: external compute, DB's, external APIs, AWS services


## CloudTrail
- CloudTrail Is a product which logs API actions/calls and account events. Eg. Stop instance, change security group, delete S3 bucket--all logged.
- Very often used to diagnose security or performance issues, or to provide quality account level traceability
- Enabled by default in AWS accounts and logs free information with a 90 day retention
- It can be configured to store data indefinitely in S3 or CloudWatch Logs

### CloudTrail - Vocab
- CloudTrail Event: When CloudTrail logs API calls/activities, it logs them as CloudTrail Events; a record of an activity in an AWS account (action taken by user, role, or service)
- Event History: Record of activity, default stored for 90 days. Enabled by default, no cost for the 90 day history
- Trail: In order to customize CloudTrail service, you need to create 1 or more "Trails". Unit of configuration within CloudTrail. Logs events for the AWS region it's created in.
-- One-Region Trail: Only EVER in the region it's created in. Only logs events for this region
-- All-Regions Trail: Collection of Trails within every AWS Region, but logged as one trail. If AWS adds new regions, they get automatically added to All Region Trails
- Global Service Events: A very small number of services log events globally to ONE region (us-east-1). Eg. IAM, STS, CloudFront. A Trail needs this enabled in order to log these events. Should be enabled in the created Trail by default

EXAM: CloudTrail is a REGIONAL Service. A Trail logs events for the AWS region it's created in

### CloudTrail - Event Types (3)
1. Management Events: Provide information about mgmt operations performed on resources in your AWS account; AKA Control Plane Operations. Eg. Creating VPC, terminating EC2 instance
2. Data Events: Contain information about resource operations performed on or in a resource. Eg. Objects loaded to s3, accessed from s3, lambda function being invoked
3. Insight Events (discussed later): Identify unusual activity, errors, or user behavior in your account
NOTE: By default, CloudTrail only logs Management Events

### CloudTrail - Trails
- One-Region Trail: Only EVER in the region it's created in. Only logs events for this region
- All-Regions Trail: Collection of Trails within every AWS Region, but logged as one trail. If AWS adds new regions, they get automatically added to All Region Trails
NOTE: Small number of services log events globally to ONE region (us-east-1). Eg. IAM, STS, CloudFront. These are called Global Service Events. So, Trails are either logged within the region they were created in, or to us-east-1 if it's a Global Service Event. Or to all regions if All-Region Trail
-- All events are captured by a Trail (management events by default, data events if manually enabled)
-- A Trail stores events in an S3 bucket by default. Logs generated and stored in an s3 bucket can be stored indefinitely; charged for storage in s3. Trail event files are small JSON files, however
-- Alternative to previous bullet, CloudTrail can integrate with CloudWatch Logs and deposit logged data there. Advantage here is being able to use CloudWatch Logs features

## CloudTrail - Continued
- Recent addition to CloudTrail, you can now create Organizational Trail. This means, if created from mgmt account, it can store all info from all accounts in the Org. Making it a single management point for all API and account events across every account in the Org.
- CloudTrail is enabled by default on AWS accounts. Free 90 day event history retention; no storage in s3 unless you configure a Trail
- Trails are how you can config s3 and CW Logs
- By default, only Management Events are logged in CloudTrail (creating, stopping, terminating instance, console login, any AWS interaction). Data events need to be enabled, but come at an extra cost

EXAM: Global Service Events: While most AWS services log data to the same region that service is in, there are a few True Global Services that are logged as GSE's in us-east-1; a Trail must be enabled to capture GSE data
EXAM: CloudTrail is NOT real-time. CT typically delivers log files within 15 minutes of the account activity occurring. CloudTrail is not the product choice you want if you need real time logging.

## CloudTrail Pricing - https://aws.amazon.com/cloudtrail/pricing/
- 90 day Event History is Free
- Get 1 copy of Management Events free in every region in each AWS account
- Beyond that, Management Events: $2.00 / 100,000 events (after the first free copy)
- Data Events: $0.10 / 100,000 events
- CloudTrail Insights $0.35 / 100,000 events analyzed

## DEMO - Implementing an Organizational Trail (CloudTrail)
This CloudTrail will be configured for all regions and set to log global services events.
We will set the trail to log to an S3 bucket and then enhance it to inject data into CloudWatch Logs.

NOTE: You can create individual trails in accounts, but it's always more efficient to use an Organizational Trail.
NOTE: By default when you create a Trail, it creates a Trail for all AWS Regions in your account

EXAM: S3 bucket names MUST be GLOBALLY UNIQUE

1. iamadmin login > CloudTrail > Trails > Create trail > Trail name "Animals4lifeOrg" > check "Enable for all accounts in my org" > uncheck Enable under "Log file SSE-KSM encryption (use this in production, not in this practice) > Enable CloudWatch Logs > CW Logs Role name "CloudTrailRoleForCloudWatchLogs_Animals4Life" > Next > select Event types to log: check Management Events > Next > Review / Create trail
- After creating, can take some time for data to start appearing
2. Open the s3 link on the new Trail > move down through CloudTrail/ > View json.gz (can decompress with Chat GPT) to verify Trail is working
3. In new tab, nav to CloudWatch > Log Groups > Log Streams all in us-east-1
- Account ID is contained within Log Stream file name.
- In Cloud Trail Event History, this will log Events for 90 days even if you have no Trail created. Trails allow you to customize what happens to the data

- In this demo, we created a Trail 1) to store data in s3 2) to put it into CloudWatch Logs

NOTE: s3 charges based on storage. If CloudTrail is enabled, the logs will accumulate possibly past Free Tier
- To Avoid logging events and incurring charges: Go to Trails, "Stop Logging"


## AWS Control Tower
AWS Control Tower offers a straightforward way to set up and govern an AWS multi-account environment, following prescriptive best practices. AWS Control Tower orchestrates the capabilities of several other AWS services, including AWS Organizations, AWS Service Catalog, and AWS IAM Identity Center (successor to AWS Single Sign-On), to build a landing zone in less than an hour. Resources are set up and managed on your behalf.

AWS Control Tower orchestration extends the capabilities of AWS Organizations. To help keep your organizations and accounts from drift, which is divergence from best practices, AWS Control Tower applies preventive and detective controls (guardrails). For example, you can use guardrails to help ensure that security logs and necessary cross-account access permissions are created, and not altered.
- Think of Control Tower as another evolution of AWS Organizations

### CT: Parts of Control Tower
- Landing Zone: Multi-account environment (what most people interact with). SSO/ID Fed, Centralized Logging, Auditing
- Guard Rails: Detect/Mandate rules/standards across all accounts
- Account Factory: Automates and standardizes new account creation
- Dashboard: Single page oversight of entire environment/org

#### CT: Landing Zone
A well-architected multi-account environment. Home Region is the one you deploy into and is always available.
- Built with AWS Orgz, Config, CloudFormation
- Security Organizational Unit (OU): Log Archive and Audit Accounts (CloudTrail / Config Logs)
- Sandbox OU: For testing and for less rigid security situations
- IAM ID Center (prev. AWS SSO): SSO, multiple accounts, ID Federation (using on-premise identity stores)
- Monitoring and Notifications: CloudWatch and SNS
- Service Catalog: End User account provisioning

#### CT: Guard Rails
- Rules for multi-account governance.
- Three types: Mandatory, Strongly Recommended, Elective
- GR functions in 2 Ways:
-- Preventative: Stop you from doing things (AWS Org SCP). Enforced or not enabled. Eg. Allow/deny regions, disallow bucket policy changes (prevent things from happening)
-- Detective: Compliance checks (AWS Config Rules) for maintaining Best Practices. Types: Clear, In Violation, Not Enabled. Eg. Detect / confirm if CloudTrail has been enabled on  your account (identify things happening)

#### CT: Account Factory
- Automated account provisioning. Cloud Admins or End Users (w/ appropriate permissions)
- Guardrails automatically added
- Give admin permissions to a named user (IAM ID)
- Standard Account & Network standard configuration. Eg. IP addressing using VPC
- Allows accounts to be closed or repurposed
- Can be fully integrated with a business's Software Development Lifecycle

## AWS Control Tower (continued)
- Under Control Tower, AWS Organizations creates two Organizational Units (OU): Foundational/Security and Sandbox OU's
- Within each Foundational OU and Security OU, two accounts are created in each: Audit and Log Archive accounts
- Account Factory: Create/configure/delete accounts using Templates: Account Baseline (template) and Network Baseline (template)
- For Guardrails, AWS uses AWS Congfig and SCP
- Drift: Divergence from best practices

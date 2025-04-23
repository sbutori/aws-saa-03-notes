# CONTAINERS AND ECS

## Introduction to Containers
Another type of compute, container computing. Reminder, Virtualization is multiple O/S's operating on a single collection of hardware (guest O/S's). Containerization improves on Virtualization by aggregating and avoiding duplicating O/S's on different Virtual Machines in a system.
- With containerization, you have Host Hardware at the bottom, Host O/S on that, then a Container Engine (think Docker) atop the Host O/S. Now apps/whatever can run within isolated containers in the Container Engine. Each app no longer requires a full O/S, so they run lighter and allow more apps running on single piece of hardware
- Container is a running copy of a Docker Image, a Docker Container adds read/write capability to Docker Image. Similar to how an EC2 instance is a running copy of an EBS Volume. Docker Images are set up in layers which are read only, changes to layers are tracked using differential architecture (only tracks changes from 1 layer to another)
- Collections of Container Images are kep in a Container Registry Eg. Docker Hub
- Dockerfiles used to create Docker Images
- Great substitute for VMs if you don't need a full O/S for the container

## DEMO - Creating 'container of cats' Docker Image
Create a docker image containing the 'container of cats' application.
- Prereq: Create a DockerHub account
1. 1 Click Deploy.
2. Nav to new EC2 instance > Connect > Session Manager
3. Commands https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0030-aws-associate-ec2docker/lesson_commands.txt
- `sudo amazon-linux-extras install docker` - Install Docker
- `sudo service docker start` - Start Docker
- `docker ps` - Test Docker. Expect a permissions error
- `sudo usermod -a -G docker ec2-user` - Add permissions
- `exit` and reopen a Session Manager
- `sudo su - ec2-user` - Log in as EC2 user
- `docker ps` now works
- `cd container`, `ls -la`
- `docker build -t containerofcats .` - Build container image
- `docker images --filter reference=containerofcats` - Show images in this container
- `docker run -t -i -p 80:80 containerofcats` - Map port 80 on container to port 80 on ec2 instance with cats image
- Log in to docker hub
docker login --username=YOUR_USER
docker images
docker tag IMAGEID YOUR_USER/containerofcats
docker push YOUR_USER/containerofcats:latest


## ECS - Elastic Container Service - Concepts
ECS is to containers as EC2 is to virtual machines. A managed container-based compute service
- Runs in two modes 1. EC2 2. Fargate
- ECS let's you create a cluster. A Cluster is where you container runs from.
- Elastic Container Registry is the ECS Container Registry
- Container Definition: Defines Images and Ports
- Task Definition: A self contained application; the app as a whole. Stores a Task Role, an IAM Role.
- Task Role: IAM Role which the TASK assumes

EXAM: Task Roles are the best practice way giving containers within ECS permission to interact with AWS Resources
- ECS Service. Config'd with Service Definition: Tells how a task will scale and determines high availability. Can use load balancer. Restarts.
EXAM: Cluster Modes available within ECS: Network Only (Fargate), EC2 Linux + Networking, EC2 Windows + Networking
EXAM: Benefits of containers: Fast to start up, Portable, Lightweight

## ECS - Cluster Mode
ECS is capable of running in EC2 mode or Fargate mode. EC2 mode deploys EC2 instances into your AWS account which can be used to deploy tasks and services.
- With EC2 mode you pay for the EC2 instances regardless of container usage.
- Fargate mode uses shared AWS infrastructure, and ENI's which are injected into your VPC.
- You pay only for container resources used while they are running.


The Two Modes when running ECS: EC2 Mode, Fargate Mode
- Main differentiator: What you manage VS what AWS manages

### EC2 Mode
- Creates EC2 instances as Container Hosts; You'll be paying for these Hosts/Instances
- EC2 Mode for when you want to use containers in your infrastructure but you NEED to manage container host capacity and availability

### Fargate Mode
- No management of EC2 instances as Container Hosts; no servers to manage.
- Not paying for EC2 instances.
- The main difference is how containers are Hosted: Fargate Shared Infrastructure instead of EC2 instances.
-- Tasks / Services run from the Shared Insfrastructure and are injected in
- Only pay for the container Resources that you consume

EXAM: When to use EC2 vs ECS (EC2) vs Fargate
- EC2 VS ECS. Containers? ECS. Containers great for isolating applications
- EC2? Large consistent workload, price conscious, make use of existing reservations
- Overhead conscious? Fargate. Less mgmt overhead.
- Small / Burst / Batch / Periodic workloads? Fargate (since you only pay for consumed resources)

## DEMO ECS - Deploying 'container of cats' using Fargate
Create a Fargate Cluster, create a task and container definition and deploy the world renowned 'container of cats' Application from Dockerhub into Fargate.

1. Create Cluster: ECS Console > Clusters > Create Cluster > Networking Only option > name "allthecats", (don't tick New VPC box, use Default VPC instead by not clicking) > Create. If you get ECS error, redo these steps
2. Create Task Definition (deploy container): Task Definitions > New > Compatibility: Fargate > Next Step > def name "containerofcats", no task role needed, operating system family: linux, task size: 1bg memory, 0.5vCPU > click "Add Container": name "containerofcatsweb", image "docker.io/acantril/containerofcats", Memory Limit Soft 1024, Port Mappings: 80 tcp > Add
3. Run it: Clusters > allthecats > Tasks tab > Launch type: fargate, OS family: linux, select VPC: Cluster VPC: default selection, subnet: select 2 at random > Create/Add
4. Cleanup > Stop Task Definition > Task: Actions: Deregister > Cluster: allthecats: Delete


## ECR - Elastic Container Registry
Like Docker Hub but the AWS version. A managed Container Image Registry service
- Each AWS account has a public and private registry and can have many repositories
-- Public = Public Read only, R/W requires permissions
-- Private = Permissions required for any R/O or R/W
- In each repo you can have many container images
- Images can have several tags

### Benefits of ECR
- Integreated with IAM for Permissions
- Offers security scannings, Basic and Enhanced Scanning (enh. uses Inspector product)
- Near real-time metrics delivered into CloudWatch
- Logs API actions into CloudTrail
- Generates events deliverd to EventBridge
- Offers Replication (cross region and cross account)

## Kubernetes - K8
Open source container orchestration system. Cloud agnostic product - can use on many platforms
- Cluster: Highly available cluster of compute resources
-- Vocab: Cluster Control Plane, Cluster Nodes, containerd, kubelet, Kubernetes API
-- The control plane orchestrates containerized applications which run on nodes
- Pods: Smallest unit of computing in K8's, often seeing one-container-one-pod. Pods are NON-PERMANENT; view them as temporary things that are created for job and gone when job is done
- Cluster: A deployment of K8's
- Node: Resources within cluster; pods are placed on nodes to run
- Pods: Smallest unit in K8s, often 1 container to 1 pod
- Service; Abstraction running on 1 or more pods; what you'll understand as an application
- Job: ad-hoc, creates one or more pods until completion
- Ingress: How something external to the cluster can access the service
- Ingress Controller: Used to provide ingress (Eg. AWS Load Balance Controller)
Default storage in K8s is ephemeral by default, provided locally by a node. Like ISVs on EC2
- Persistent Storage (PV) : Volume whose lifecycle lives beyond any 1 pod using it

## Elastic Kubernetes Service (EKS)
Amazon Elastic Kubernetes Service (Amazon EKS) is a fully-managed, Kubernetes implementation that simplifies the process of building, securing, operating, and maintaining Kubernetes clusters on AWS.
- open-source and cloud agnostic
- K8s Control Plane scales and runs on multiple AZs
- Integrates with AWS svc's: ECR, Elastic Load Balancer, IAM, VPC
- EKS Cluster = EKS Control Plane & EKS Nodes
- etcd: the key-value store that k8's uses, is distributed across multiple AZs
- Nodes: can be Self-Managed (ec2 instance), Managed Node Groups (ec2 instance), or Fargate pods
- For Persistent Storage, EKS can use EBS, EFS, FSx Lustre, FSx for NetApp ONTAP

### EKS Resources
- https://docs.aws.amazon.com/eks/latest/userguide/eks-compute.html
- https://docs.aws.amazon.com/eks/latest/userguide/fargate.html


# ADVANCED EC2
## Bootstrapping EC2 using User Data
EC2 Bootstrapping is the process of configuring an EC2 instance to perform automated install & configuration steps 'post launch' before an instance is brought into service. With EC2 this is accomplished by passing a script via the User Data part of the Meta-data service - which is then executed by the EC2 Instance OS
- tl;dr this allows you to bring and EC2 instance up in a pre-configured state
- EC2 Bootstrapping = EC2 Build Automation
- This process uses User Data via meta-data IP (http://169.254.169.254/latest/user-data) - Note, this is instead of /meta-data
-- EC2 doesn't interpret this data, the OS needs to understand the User Data
- This boostrapping ONLY happens on Launch. If you update User Data after the fact, it won't be applied
- User Data is OPAQUE to EC2, its just a block of data to EC2
- User Data is NOT secure; don't use it for PWs or long term creds
- User Data limited to 16KB in size
- If you want to update User Data... stop instance, update data, and restart

EXAM: Boot-time-to-Service-Time: how quickly you can bring an instance into service and have it ready for use by customers
- measured in minutes

## DEMO - Bootstrapping WP Installation (parts 1 and 2)
Bootstrap two EC2 instances with WordPress and the 'cowsay' login banner customizations.
- The first, directly using User Data via the console UI
- the second, using CloudFormation

Steps:
1. 1-click deploy > Create stack
2. EC2 > Launch Instance (x2) > Name "A4L-MANUALWORDPRESS" > keypair: proceed without > Network Settings, Edit
3. Instance Network Settings: VPC, select a4l-vpc1 > subnet, select sn-web-A > Select Existing Security Group, BOOTSTRAP-[etc] > Advanced Details, User Data, paste in .txt file provided > Launch Instance
4. EC2 Instance Connect > Will see custom Banner above normal stuff if you wait long enough. Paste in the commands in the command file to see user data
Note: Within EC2 Connect, "cd /var/log" you can use the cloud-init-output.log and cloud-init.log to diagnose any bootstrapping issues
5. CloudFormation > 1-click deploy > etc etc
6. Clean up > Terminate EC2 instance > Delete CFN stack


## EC2 Instance Roles & Profile
EC2 Instance Roles/Profiles are how applications running on an EC2 instance can be given permissions to access AWS resources on your behalf. Short-term temporary creds are available via the EC2 instance Metadata and are renewed automatically by the EC2 and STS Services

Instance Role Architecture uses an IAM role that allows EC2 instance to assume it. There is an InstanceProfile wrapper that actually attached to the instance. Creds delivered within metadata
- creds used to give InstanceProfile IAM role access are inside meta data
-- iam/security-credentials/role-name
-- creds always valid/auto-rotated
- BEST PRACTICE: Always use Roles when possible, better security. CLI tools will use role creds automatically

## DEMO - Using EC2 Instance Roles
Create an EC2 Instance Role, apply it to an EC2 instance and learn how to interact with the credentials this generates within the EC2 instance metadata.

Steps:
1. 1-click deploy
2. EC2 > instance connect > EC2 Instance Connect > test command `aws s3 ls` > need creds
3. Create IAM Role > IAM > Roles > Create Role > Trust Entity, AWS service: select `EC2` > Next, Add Permissions, search `s3`, select AmazonS3ReadOnlyAccess > Next, Role name "A4LInstanceRole" > Create Role
4. Attach Role to Instance: EC2 > Right-click instance, Security: Modify IAM Role > Choose Role A4LInstanceRole
5. You can now `aws s3 ls` command in EC2 Connect because you have applied the IAM Role that has the S3ReadOnly permission to the EC2 instance
6. Clean up > IAM: Delete A4LInstanceRole > EC2: Security: Modify IAM, select NO IAM Role > CFN: Delete Demo stack

## EC2 - SSM Parameter Store
The SSM Parameter store is a service which is part of Systems Manager which allows the storage and retrieval of parameters - string, stringlist or secure string.
- The service supports encryption which integrates with KMS, versioning and can be secured using IAM.
- The service integrates natively with many AWS services - and can be accessed using the CLI/APIs from anywhere with access to the AWS Public Space Endpoints.

As previously mentioned, passing secret data through to EC2 instance with User Data is bad practice.
- SSM is storage for configuration and secrets
- Parameter store can store the following: String, StringList, SecureString
-- There are hierarchies and Versioning of parameters
-- Can be stored as plaintext and ciphertext (integrating with KMS)
- Public Parameters are available, like the latest AMIs per region

## DEMO - Parameter Store
Create some Parameters in the Parameter Store and interact with them via the command line - using individual parameter operations and accessing via paths.

Steps:
1. Nav to Systems Manager in AWS Console > Parameter Store > Create parameter
2. Parameter 1 details: Name "/my-cat-app/dbstring" > value "db.allthecats.com:3306" > description "Connection string for cat app" > create
- the fwd slashes create hierarchy in Parameter Store
3. Parameter 2: Name "/my-cat-app/dbuser" > value: "bosscat" > create
4. Param 3 (secure): Name "/my-cat-app/dbpassword" > Type: SecureString > value: amazingsecretpassword1337 (encrypted)
5. Param 4: Name "/my-dog-app/dbstring" > value: "db.ifwereallymusthavedogs.com:3306"
6. Param 5: Name "/rate-my-lizard/dbstring" > value "db.thisisprettyrandom.com:3306"
7. CloudShell (top menu) > try these commands:
aws ssm get-parameters --names /rate-my-lizard/dbstring
aws ssm get-parameters --names /my-dog-app/dbstring
aws ssm get-parameters --names /my-cat-app/dbstring
aws ssm get-parameters-by-path --path /my-cat-app/
aws ssm get-parameters-by-path --path /my-cat-app/ --with-decryption
8. Clean up: Select all params > Delete

## System and Application Logging on EC2
CloudWatch is for metrics*
CloudWatch Logs is for logging and interpreting that logged data*
* Neither of these natively capture data inside an instance
- CloudWatch Agent is required for CW/CW Logs to natively capture data in an instance (plus config and permissions)
-- Configured / Permitted CW Agent pulls logs from Instance and sends to CW / CW Logs
-- The permissions the Agent needs will use an IAM Role

## DEMO - Logging and Metrics with CloudWatch Agent
Download and install the CloudWatch Agent and configure it to capture 3 log files from an EC2 instance: /var/log/secure, /var/log/httpd/access_log, /var/log/httpd/error_log

Steps:
1. 1-click deploy
2. Connect to instance: EC2 instance connect
3. Install CW Agent to Instance connect with `wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm`
4. Use IAM to create Role/Permissions for Agent: IAM > Roles > Create Role > select options: AWS service/EC2 > select (2) policies `CloudWatchAgentServerPolicy` And `AmazonSSMFullAccess` > next > name "CloudWatchRole" > Create Role
5. Connect Role to Instance: EC2 Instances > right-click instance, Security, Modify IAM role > Select CloudWatchRole > Update IAM Role
6. EC2 Connect to Start CW Agent config wizard: Default OS, Enter > Default region (EC2), enter > user: root, enter > enter statsd, enter statsd port > interval, enter > enter for everything until "Which default metrics config do you want? input `3` for Advanced... [FOLLOW VIDEO for rest of this crap, it's mostly pressing Enter for defaults] - https://learn.cantrill.io/courses/1820301/lectures/41301605
7. Store Config in Parameter Store: Press enter more for Defaults. Note: Most all of it is defaults, pay attn to the few that aren't
8. CloudWatch Console > Log Groups. Look for the three file paths you created
9. Cleanup > EC2, security: remove CW Role > IAM, Roles, Delete role > CFN, delete Stack

## EC2 Placement Groups
Allows you to choose placement of EC2 instances, so you can ensure that they're physically close together or not.

3 placement groups available within AWS:
1. Cluster (PERFORMANCE)
2. Spread (Resilience)
3. Partition (Topology Awareness)

- Cluster. For highest EC2 performance. Launch these all at the same time for similar placement/capacity, will be launched into single AZ. Instances in Cluster group have fast direct connections to each other. Cluster achieves 1-Gbps per single stream instead of the standard 5Gbps/stream. Lowest latency and max Packets-per-second possible in AWS (when paired with Enhanced Networking)
EXAM: CLUSTER Placement Groups are ONE AZ ONLY, which is locked when launching the first instance. Close prox is what gives them best Performance of placement groups
EXAM: VPC peers in cluster placement groups can span AZs, but this impacts performance
EXAM: Cluster PGs not supported on all instance types
EXAM: Launch all instances within cluster group at same time as best practice
EXAM: 10Gbps single stream performance; highest performance, lowest latency, best throughput

- Spread. For max availability and resilience for an application; span AZs to accomplish this. Use case: small number of critical instances that need to be kept separate from each other
EXAM: Spread PG instance LIMIT: 7 INSTANCES per AZ (HARD limit)
EXAM: Spread PGs provide "infrastructure isolation" to provide max availability and resilience
EXAM: Each instance runs from a different rack, each rack has its own network and power source
EXAM: Not supported for Dedicated Instances or Hosts

- Partition. Designed for when you have more than 7 instances per AZ but you still need to be able to separate those instances into separate fault domains. Designed for huge-scale parallel processing systems
EXAM: For when you need max availability/resilience but MORE THAN 7 instances per AZ
EXAM: Max 7 PARTITIONS per AZ, but each partition can contain more than 7 instances
EXAM: Each partition has its own rack
EXAM: Instances can be placed into a specific partition or can be auto-placed
EXAM: for 'TOPOLOGY' aware applications; Eg: HDFS, HBase, Cassandra

## EC2 - Dedicated Hosts
It's an EC2 Host dedicated only to you. Dedicated hosts are EC2 Hosts which support a certain type of instance which are dedicated to your account. Eg: a1, c5, m5
- Billing: No EC2 instance charges, you pay for the host
- You can pay an on-demand or reserved price for the hosts and then you have no EC2 instance pricing to pay for instances running on these dedicated hosts.
- Generally dedicated hosts are used for applications which use physical core/socket licensing

### How they work: Host is designed for a specific family and size of instance. Eg. a1 dedicated host has 1 socket and 16 cores

### Dedicated Host Limitations (things that aren't supported)
- AMI limits - Not supported: RHEL, SUSE Linux, Windows AMIs
- Cannot used Amazon RDS instances
- Placement Groups not supported

Note: Using RAM (resource access manageR) product, hosts CAN BE shared with other ORG accounts

## EC2 - Enhanced Networking & EBS Optimized
Enhanced networking is the AWS implementation of SR-IOV, a standard allowing a physical host network card to present many logical devices which can be directly utilized by instances.
- This means lower host CPU usage, better throughput, lower and consistent latency
- EBS optimisation on instances means dedicated bandwidth for storage networking - separate from data networking.
- Resource: https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enhanced-networking.html

### Enhanced Networking
A feature designed to improve overall performance of EC2 networking--required for high performance things like Cluster Placement Groups.
- Uses technique called SR-IOV so that the network interface inside EC2 is aware of virtualization. This all increases IO, lower host CPU usage, more bandwidth, and higher packets-per-second (PPS), consistent lower latency

### EBS Optimized Instances
What we know about EBS already: EBS is block storage over the network. EBS Optimized means DEDICATED capacity for EBS


# Route 53 - Global DNS

## R53 Public Hosted Zones
### R53 Hosted Zones (HZs):
- An R53 Hosted Zone is a DNS DB for a domain
- GLOBALLY RESILIENT
- HZs are created automatically when you register a domain using R53. But HZs can be created separately
- HZs are what the DNS system references; authoritative source to confirm a domain Eg. animals4life.org
- Two types of hosted zones in R53: public and private

### Public Hosted Zones
A PUBLIC hosted zone is a container that holds information about how you want to route traffic on the internet for a specific domain which is accessible from the public internet
- Creating a Public HZ creates 4 R53 Nameservers that are all accessible from public internet and VPCs
- Monthly cost for hosting public HZ and small cost for queries against it

## R53: Private Hosted Zones
A private hosted zone is a container that holds information about how you want Route 53 to respond to DNS queries for a domain and its subdomains within one or more VPCs that you create with the Amazon VPC service
- Private Hosted Zones are ALWAYS associated with a VPC and are only accessible within those VPCs
- Enabled split-view (overlap public / private) for PUBLIC and INTERNAL use with the same zone name (eg. only intranet for internals but see same thing via public route if external)
- To gain access to a private hosted zone, you need to be querying from within a VPC, AND that VPC needs to be explicity associated with the private host

## R53: CNAME vs R53 Alias
The issue with only using CNAMEs: `A` record maps a NAME to IP Address ie. catagram.io => 1.3.3.7. CNAME on the other hands, maps NAME to NAME (ie. www.catagram.io => catagram.io)
- Problem: can't use a CNAME for the apex of a domain, also known as the naked domain. AWS services like ELB only use DNS names, so just a CNAME like catagram.io would be invalid within ELB
- Solution: R53 Alias records let you iuse CNAME for domain apex

### R53 Alias
- ALIAS records map a NAME to an AWS Resource
- ALIAS records can be used for naked/apex and normal records. Alias functions like CNAME for non-apex/non-naked domains
- No charge for ALIAS requests pointing at AWS resources
EXAM: For AWS Services, ALIAS should be your default choice

## R53: Simple Routing
Simple routing lets you configure standard DNS records, with no special Route 53 routing such as weighted or latency. With simple routing, you typically route traffic to a single resource, for example, to a web server for your website.
- Simple Routing supports 1 record per name (www)
- Each record can have multiple values
- Simple Routing does not support health checks
EXAM: Use Simple Royuting when you want to route requests towards ONE SERVICE such as a web server

## R53 Health Checks
Amazon Route 53 health checks monitor the health and performance of your web applications, web servers, and other resources.
- Each health check that you create can monitor one of the following:
-- The health of a specified resource, such as a web server
-- The status of other health checks
-- The status of an Amazon CloudWatch alarm

- Health checkers are located GLOBALLY
- Health Checks occur every 30s (or every 10s if you pay more)
- States: Healthy or Unhealthy
- Globally, you have distributed health checkers. If 18%+ of health checkers report status as healthy, the health check returns healthy
- In most cases, an UNHEALTHY record is NOT returned in queries

## R53 - Failover Routing
Failover routing lets you route traffic to a resource when the resource is healthy or to a different resource when the first resource is unhealthy
- Use when you want to configure "active passive failover"

# DEMO - Using R53 and Failover Routing - NOTE: This demo requires an R53 registered domain
Create Failover routing and private hosted zones. https://learn.cantrill.io/courses/1820301/lectures/41301585

## R53 - Multi Value Routing
Multivalue value routing lets you configure Amazon Route 53 to return multiple values, such as IP addresses for your web servers, in response to DNS queries. You can specify multiple values for almost any record, but multivalue answer routing also lets you check the health of each resource, so Route 53 returns only values for healthy resources
- Improves availability, but NOT a replacement for load balancing
- Up to 8 healthy records are returned. If you have more than 8 records, 8 at random are returned

## R53 - Weighted Routing
Weighted routing lets you associate multiple resources with a single domain name (catagram.io) and choose how much traffic is routed to each resource. This can be useful for a variety of purposes, including load balancing and testing new versions of software.
- Can assign weights to each hosted zone. Eg. 3 Hosted Zones, 40, 40, 20, totalling 100. First 2 each get 40% of traffic, last only gets 20% of traffic. This doesn't need to add to 100
- A 0 weight means record is never returned unless all are 0, then all are considered

## R53 - Latency Routing
Should be used when trying to optimize for performance and user experience
- Latency-based routing supports one record with the same name in each region
- In the background, AWS mantains a latency table between users and regions and matched user to lowest latency
- Latency DB is NOT real time

## R53 - Geolocation Routing
Geolocation routing lets you choose the resources that serve your traffic based on the geographic location of your users, meaning the location that DNS queries originate from.
- Geolocation records are tagged with a location
- Does not return the closest record, but that which are applicatble or the default or no answer. "Default" is backup answer
- Good for regional restrictions, language specific content, or laod balancing across regional endpoints
- To route traffic based on customer location. Not proximity, but based on tag and specificity
EXAM: Geolocation routing NOT about closest recoird, but returns RELEVANT locations only

## R53 - Geoproximity Routing
Geoproximity routing lets Amazon Route 53 route traffic to your resources based on the geographic location of your users and your resources.
- You can also optionally choose to route more traffic or less to a given resource by specifying a value, known as a bias.
-- A BIAS expands or shrinks the size of the geographic region from which traffic is routed to a resource. Can define a plus or minus bias + / -

## R53 Interoperability
How Route53 provides Registrar and DNS Hosting features and steps through architectures where it is used for BOTH, or only one of those functions - and how it integrates with other registrars or DNS hosting.

R53 does two things: 1) Domain Registrar 2) Domain Hosting. It can do both or either, user choice

### R53 - When you REGISTER domain [Domain Hosting / Domain Registrar]
1. Accepts your money (domain registration fee) [DR]
2. Allocates 4 name servers (NS) [DH]
3. Creates a Zone File (domain hosting) on the above NS's [DH]
4. R53 communicates with registry of TLD (domain registrar) [DR]

## R53 - DNSSEC
DNSSEC strengthens authentication in DNS using digital signatures based on public key cryptography. With DNSSEC, it's not DNS queries and responses themselves that are cryptographically signed, but rather DNS data itself is signed by the owner of the data.


# Relational Database Service (RDS)
## Database Refresher & Models
DBs are systems which store and manage data:
DB Types:
1) Relational Database Management System (RDBS, commonly SQL)
- RDBS has a structure in & between tables of data; rigid schema
- Fixed relationship between tables. These relationships are defined in advance
2) Non-relational (NoSQL); more relaxed schemas
- Common Examples of Non-relational DBs:
-- Key-value DB's: just key/value pairs and no schema otherwise. Highly scalable and fast. Good for in-memory caching
-- Wide Column Store: Every item in table has same key layout (Partition Key, + other key). Items in a table can have differing attributes. Eg. DynamoDB
-- Document Database. Good for DBs of orders or contact list. Key/Value but the value is the accessible document+data
-- Column Store (AKA OLTP, good for transactional processes like collections of orders) and Row Store (RedShift; good for analytics)
-- Graph. Data and the relationships of data stored in DB

## RDS - ACID vs BASE
This lesson steps through the ACID and BASE Database transaction models and introduces the CAP Theorem
- transaction models AKA define a few things about transactions to and from a DB
- CAP Theorem - Consistency, Availability, Partition Tolerant (resilience). CAP Theorem says that any DB product only capable of delivering a max of TWO of these factors
-- ACID focuses on consistency, BASE focuses on Availability

### ACID - atomic, consistent, isolated, durable
EXAM: 'ACID' mentioned in exam? RDS based DB. Acid limits DB scalability
- Atomic: either all or NO components of a transaction succeed or fail
- Consistent: transactions move DB from one VALID state to another; nothing in-between is allowed
- Isolated: if multiple transactions occur at once, they don't interfere with each other
- Durable: once committed, transactions are durable; stored on non-volatile memory and are resilient to outages/crashes

### BASE - Basically available, soft state, eventually consistent
- Basically Available: Read / Write ops are available 'as much as possible' but without any consistency guarantees
- Soft State: The DB doesn't enforce consistency, this is offloaded onto the application/user
- Eventually Consistent: If we wait long enough, reads from the system will be consistent
BASE = highly scalable, Eg DynamoDB
EXAM: BASE = noSQL

## RDS - Databases on EC2
NOTE: Running DB on EC2 is generally bad practice as there is usually a better alternative
Why you might run DB on EC2...
- If you need access to the DB instance OS
- Advanced DB Option tuning (DBROOT)
- Vendor/decision maker demands
- Need to run DB or DB Version AWS doesn't provide
- Need a specific OS/DB combo or architecture AWS doesn't provide

Why you SHOULD NOT run DB on EC2
- Admin overhead is high to manage on EC2 and DBHost
- Backups / disaster recovery management becomes more difficult
- EC2 is a single AZ. If zone fails, DB access fails
- Features. AWS DB products are great and you could be missing out
- EC2 is ON or OFF. No serverless easy scaling
- Replication.
- Performance. AWS invests time into optimisation / features that you're missing

## Demo - RDS - Splitting Wordpress Monolith => APP & DB
- Moving DB to new AZ, separate from Webserver/App

Steps:
1. 1 click deploy > Create complete
2. Copy WP instance IP, paste to browser > Title "The Best Cats!!", user "admin", PW "an1m4ls4l1f3", email test@test.com > Install
3. A4L-WordPress, ec2 connect `mysqldump -u root -p a4lwordpress > a4lwordpress.sql` > PW an1m4ls4l1f3
4. Inject a4lwordpress.sql into new DB `mysql -h privateipof_a4l-mariadb -u a4lwordpress -p a4lwordpress < a4lwordpress.sql`
5.... Error at step 4.
6. Clean up > CFN > Delete Stack

## RDS - Architecture
- Known as "DBaaS", but more accurately RDS is a DBSaaS "DB Server as a Service"
- RDS provides multiple databases on one DB Server (instance)
- RDS is a managed service, so no access to OS or SSH Access
- RDS operates within a VPC
- RDS Instance can have more than 1 DB in it, and each RDS Instance has its own Dedicated EBS-provided storage
NOTE: Amazon Aurora is DIFFERENT from RDS
Cost: Billed for resource allocation
-- Cost #1: instance size/type
-- Cost #2: Multi-AZ or not
-- Cost #3: Storage type/amount
-- Cost #4: Data transferred
-- Cost #5: Backups & Snapshots
-- Cost #6: Licensing

## RDS - DEMO - Migrating EC2 DB into RDS
Create a MySQL RDS instance and migrate the Wordpress Database from the self-managed MariaDB server running on EC2 into this RDS instance.

Steps:
1. 1 click deploy
2. Set up website: EC2 > A4L-Wordpress instance, copy IPv4 address, paste into browser > title "The Best Cats!!", username "admin", PW "4n1m4ls4L1f3", email "test@example.com" > install WP
3. Set Up data to move: Blog Post > Posts > delete Hello World > Add New > H1 "The Best Cats Ever!!" > add Gallery widget and upload zipped file, Publish
4. Set up RDS DB: RDS > Subnet Groups > Create a DB Subnet Group "a4lsngroup" for title/descrip > VPL a4l > Add subnets, select us-east-1a/b/c > Pick your subnets > go to VPC, Subnets and look for full names of sn-db-A/B/C to select those ones > Create
5. Provision DB: RDS > DB > Create DB > Standard Create > select MySQL > engine version (currently 8.0.32 in demo video) > Template, Free Tier > DB instance identifier "a4lwordpress" > admin "a4lwordpress" > PW "4n1m4ls4L1f3" > Storage, uncheck "Enable storage autoscaling" > Connectivity, select a4l-vpc1 > VPC security group (firewall), Create New > name "a4lvpc-rds-sg" > Add'l Configuration, Initial DB Name "a4lwordpress" > Create DB
6. Config Security Group so RDS DB is connected: RDS > DB's > a4lwordpress > Connecitivity & Security tab, click the VPC SG link and open in new tab > find your SG > Inboud Rules > Edit > Add Rule, Type MYSQL/Aurora, Source (magnifying glass icon) "MIGRATE2RDS-Instance..." > Save Rule
7. Migrate Data to new RDS DB, create backup file to move: EC2 Connect with A4L-Wordpress >
`mysqldump -h PRIVATEIPOFMARIADBINSTANCE -u a4lwordpress -p a4lwordpress > a4lwordpress.sql` -> replace private IP with Private IPv4 of a4l DB instance > Enter > use PW above > Enter
8. Move backup SQL DB instance into RDS: `mysql -h CNAMEOFRDSINSTANCE -u a4lwordpress -p a4lwordpress < a4lwordpress.sql` -> Replace CNAME with Endpoint name in RDS db instance > Enter
9. Point WP at RDS instance:
`cd /var/www/html` >
`sudo nano wp-config.php` -> Go to Database Hostname and replace the IPv4 pointing at the WB DB EC2 instance with the Endpoint name of the RDS instance...
Looks like...
/** Database hostname */
define( 'DB_HOST', 'a4lwordpress.cloy7bpdvh9v.us-east-1.rds.amazonaws.com' ); > ctrl+o to Save, ctrl+x to Exit nano
10... Demo broke at this stage
11. Cleanup: RDS > Delete DB > VPC > Delete Security group "a4lvpc-rds-sg" > Cloufformation > Delete Stack

## Relational Database Service (RDS) MultiAZ - Instance and Cluster Deployments
MultiAZ is a feature of RDS which provisions a HIGHLY AVAILABLE instance set.
- The product provides MultiAZ instance where a standby replica is kept in sync Synchronously with the primary instance.
-- The standby replica cannot be used for any performance scaling, only availability.
- MultiAZ cluster mode: where a write and two reader instances are kept in sync Synchronously. The reader instances can be used for read operations allowing for limited read scaling.
- Backups, software updates and restarts can take advantage of MultiAZ to reduce user disruption.

## MultiAZ Instance Deployments
- Synchronous replication of the primary DB to the standby DB
- Standby is only for backups and is never accessed unless a failover is required. Failover can take 60-120s
- Not free
- Only ONE standby replica due to architecture
- Same region only, but different AZs in region
- Backups taken from Standby to improve Primary performance

## MultiAZ Cluster Deployments
NOTE: Keep in mind the differences between MultiAZ Cluster Deployments and Amazon Aurora
- ONE Writer replicates to TWO reader instances, all in different AZs. With Aurora, you can have more than 2 reader instances
- These Readers (called Standby in Instance Deployment) can be used for Read operations, allowing for some read scaling
- Faster Hardware than Instance Deployment
- Faster failover, ~35s
- Writes considered "committed" when 1 of the readers has confirmed it

## RDS Automatic Backup, RDS Snapshots and Restore
- RDS is capable of performing Manual Snapshots and Automatic backups
-- Manual snapshots are performed manually and live past the termination of an RDS instance
-- Automatic backups can be taken of an RDS instance with a 0 (Disabled) to 35 Day retention.
- Automatic backups also use S3 for storing transaction logs every 5 minutes - allowing for point in time recovery.
- Snapshots can be restored .. but create a new RDS instance.
EXAM: RDS Backups live in S3 but are AWS Managed, so you never seem them

### RDS Backup - Snapshots (Manual)
- First snapshot is full copy, after that it's incremental copies
- Snapshots DON'T expire, they live beyond the termination of an RDS instance -- you have to delete them manually
- Technically, you could reach a 5 minute Recovery Point Objective

### RDS Backup - Automated Backups
- Occur once per day
- Think of them as automated snapshots
- If using single AZ, plan for a small IO pause and do it during downtime
- Every 5 mins, Transaction Logs are recorded to S3. With these plus snapshots; you have a 5 Minute Recovery Point Objective
- Retention Period. 0 - 35 days before AWS deletes the automated backups. If you select 35 days, you can restore to any point in time over that 35 day period
- To avoid losing data beyond 35 days, you need to do a final manual snapshot

### RDS Backups - Cross-Region Replication
- RDS can replicate backups to another region (both snapshots and transaction logs)
- Charges apply for the cross-region data copy and for the storage in the destination region
- This feature must be enabled

### RDS Backsup - Restores
- Creates a new RDS instance when you restore an automated backup or manual snapshot
- Snapshot restore is a single point in time, at the time of creation of the snapshot
- Automated backups are restorable to any 5 minute point in time

## RDS Read Replicas
RDS Read Replicas can be added to an RDS Instance - 5 direct per primary instance.
- READ-ONLY
- They can be in the same region, or cross-region replicas.
- They provide read performance scaling for the instance, but also offer low RTO recovery for any instance failure issues
- They don't help with data corruption as the corruption will be replicated to the RR
- Asynchronous replication
EXAM: Synchronous is MultiAZ, asynchronous is Read Replicas

### Read Replicas - Why do they matter?
- Read performance and read scaling.
-- You get 5 direct read-replicas per DB instance, each providing add'l instance of read performance
-- Read-replicas can have read-replicas, but then lag starts to be a problem
-- Global performance improvements
- Recovery Point and Recovery Time Objective benefits
-- Snapshots/Backups improve RPO, with RR's offering a near 0 RPO (little potential for data loss)
-- RR's have low RTO. NOTE: For Failure only, not corruption. The corruption is also likely replicated
- Read-replicas are Read-only until promoted (until activated), then they become a normal RDS instance
- Since you can cross-region replicate, you get Global Resilience

## Demo - RDS - MultiAZ & Snapshot Restore with RDS
Working with RDS Multi AZ mode and snapshot restores

Steps:
1. 1 click deploy
2. Set up WP (The Best Cats!!, admin, 4n1m4ls4L1f3, test@test.com), create blog post
3. Create snapshot: RDS > select DB > Actions, Take snapshot > name "a4lwordpress-with-cat-post-mysql-8032" > Take snapshot
4. Enable MultiAZ: RDS > select database > Modify > Enable MultiAZ (this part costs money so I'm not doing it)

Demo 2 - Restore RDS in the case of data corruption
Steps:
1. Change WP blog post text to simulate 'corrupted' data
2. RDS > Snapshots > select snap, Actions, Restore Snapshot > name "a4lwordpress-restore" > select burstable somethings > select RDSsecurity for SG > Create/Restore
NOTE: Restoring RDS from snapshot creates a NEW instance
3. Point to new DB: EC2 connect > cd /var/www/html >
`sudo nano wp-config.php` >
Get new restored RDS endpoint and prepare to paste it into DB hostname in nano > paste into correct spot > ctrl+o to save/write > ctrl+x to exit
4. Cleanup > Delete restore snapshot > CFN delete stack

## RDS - Security
- In transit encryption (SSL/TLS) available for RDS and can be set to Mandatory on a per user basis
- At Rest encrpytion available via KMS and EBS encryption
- Transparent Data Encryption: Encryption handled within the database engine
-- RDS Oracle supports TDE using CloudHSM (even more secure as the key management is customer now AWS)

### RDS - Security - IAM Authentication
You can configure RDS to use IAM user authentication against a DB
- AUTHORIZATION is still handled internally by DB engine
- Storage, logs, snapshots, and replicas all encrypted
- Encryption can't be removed

## Amazon RDS Custom
Amazon RDS Custom is a managed database service for applications that require customization of the underlying operating system and database environment.
- fills the gap between RDS and EC2 running a DB engine
- RDS is fully managed, so OS/engine access is limited. DB on EC2 is self-managed, but has overhead. RDS Custom bridges this gap
- RDS Custom works for MS SQL and Oracle
- Can connect using SSH, RDP, Session Manager
- Customizing RDS Custom? Make sure to look at RDS DB Automation settings to ensure no disruptions (pause automation, customize, restart automation)

## Aurora Architecture
Aurora [Provisioned] is a AWS designed relational database engine officially part of RDS. Aurora implements a number of radical design changes which offer significant performance and feature improvements over other RDS database engines.
- Uses a "cluster", which other RDS engines don't have. Cluster contains a primary instance + 0 or more replicas which have read capability during normal operation
- For storage, Aurora uses a Cluster Volume, not local storage; faster provisioning, higher availability, better performance
- Max cluster volume space of 128 TiB
- Can have up to 15 Replicas
- Storage is SSD based; high IOPS, low latency

### Aurora - Billing
- "High watermark billing" -- If you scale up to 50 TiB and then scale down to 40, you'll still be paying for the full 50. Freed up storage can be re-used
- No free-tier option
- Aurora does not support micro-instances. Beyond RDS SingleAZ (micro), Aurora offers better value
- 100% Database size in backups are included
- Backup restores create a new cluster
- Backtrack: A rollback feature that lets you roll back to your existing cluster but to a previous point in time. "in place rewinds"

## Aurora Serverless
Serverless VS Provisioned
Aurora Serverless provides a version of the Aurora DB product where you don't need to statically provision DB instances of a certain size or worry about managing DB instances
- Removes the overhead of managing individual DB instances
- Uses concept of Arurora Capacity Units (ACUs)
- Serverless cluster has a min and max ACU tht you choose, can go to 0 and be paused, cluster adjusts based on load
- Same resilience as Provisioned (6 copies across AZs)

### Aurora Serverless - Uses
- For infrequently used applications
- New applications (where you're unsure of load levels)
- For variable workloads / unpredictable workloads
- For dev and test databases (since it can pause itself)
- Multi-tenant applications (like subscribing to app), where load is directed related to customer use/revenue

## DEMO - Migrating to Aurora Serverless. Advised to not actually do per tutorial, just watch

## Aurora Global Database
Aurora global databases are a feature of Aurora Provisioned clusters which allow data to be replicated globally providing significant RPO and RTO improvements for business continuity and disaster recovery planning. Additionally, global databases can provide performance improvements for customers .. with data being located closer to them, in a read-only form.
- Global level Aurora, primary region and 5 secondary region. Secondary clusters are read-only. Each secondary region has up to 16 replicas
EXAM: Great for Cross-Region Disaster Recovery and Business Continuity. ~1s replication times
EXAM: Global Read Scaling: Low latency performance improvements to global customers
EXAM: ~1s replication or less between regions. Recovery Point Objective (RPO) of 1 second and a Recovery Time Objective (RTO) of less than 1 minute

## Aurora Provisioned - Multi-master writes
Allows Aurora cluster to have multiple instances that are all capable of both reads and writes
- No lengthy failovers since replicas already have R/W capability

## RDS Proxy
Amazon RDS Proxy is a fully managed, highly available database proxy for Amazon Relational Database Service (RDS) that makes applications more scalable, more resilient to database failures, and more secure.
- Why do you want RDS Proxy? Opening/Closing connections consumes resources, takes time which creates latency. Esp true with serverless. Handling failure of DB is difficult
- RDS proxy changes your architecture by creating long term connection pools; instead of your app connecting to a DB every time its used, its connected to a proxy which are already open to a pool of DB connections
- Abstracts client away from DB failure

### RDS Proxy - When to use?
- too many connection errors esp. if using small/burst instances
- when using AWS Lambda: time saved, connection reuse, IAM auth
- Long running connections (SaaS apps) needing low latency
- Where resilience to DB failure is a priority
- To further reduce failover time
- Make DB transparent to application

### RDS Proxy - Key Facts (EXAM)
- Fully managed DB proxy for RDS/Aurora
- Auto-scaling, highly available by default
- Provides connection pooling to reduce DB load
- RDS proxy only accessibly from a VPC
- Accessed via a proxy endpoint; no app changes
- can enfore SSL/TLS
- Can reduce failover time by over 60%
- Abstracts failure away from your applications

## RDS - Database Migration Service (DMS)
- The Database Migration Service (DMS) is a managed service which allows for 0 data loss, low or 0 downtime migrations between 2 database endpoints.
- The service is capable of moving databases INTO or OUT of AWS.
- Runs using a replication instance via EC2, requiring Source/Destination endpoints and Source/Target Databases
- ONE of the endpoints must be running on AWS. Endpoints store connection info for source/target DBs
- Replication Instance runs one or more Replication Tasks
- Jobs can be 1 of 3 types:
1. Full Load (one-off full-data migration),
2. Full Load + CDC (change data capture, for ongoing replication),
3. or CDC only (to replicate only data changes, using native tools to bulk move initial data outside of DMS)
- Schema Conversion Tool assists with schema conversion
EXAM: DB Migration question? Default to DMS as answer, especially if talking about "no-downtime migration"

### DMS - Schema Conversion Tool
- Used when converting one DB engine to another
- NOT USED for moving between COMPATIBLE DB ENGINES

#### DMS and Snowball devices
- A physical device for moving data physically
- Larger migrations (multi-TB in size) over networks takes times and consumes capacity, so DMS can utilize Snowball


## Network Storage & Data Cycle - EFS (Elastic File System)
The Elastic File System (EFS) is an AWS managed implementation of NFS which allows for the creation of shared 'filesystems' which can be mounted within multi EC2 instances.
- For scalability / resiliency
- EFS is an implementation of NSFv4 (network file system version 4)
- Can be mounted in EC2 linux and shared between many EC2 instances
- EFS is private service that mounts targets inside a VPC
- Can be accessed from on-premises via VPN or AWS Direct Connect (DX)
- Uses POSIX permissions (a linux thing)
EXAM: EFS is Linux ONLY
EXAM: 2 performance modes: 1. General Purpose 2. Max I/O
EXAM: 2 throughput modes: 1. Bursting 2. Provisioned
EXAM: 2 storage classes: 1. Standard 2. Infrequent Access (IA) - S3 Lifecycle policies can be used with these

## DEMO - Implementing EFS
Implement a simple EFS file system in a VPC, configure mount targets and configure two EC2 instances to mount the file system within a mount point.

Video: https://learn.cantrill.io/courses/1820301/lectures/41301666

Steps:
1. 1 click deploy
2. Create EFS: EFS > Create File System > Customize > name "A4L-EFS", storage class Standard, disable Encryption of data at rest, pPrformance Settings: Bursting > Next
3. EFS Network Settings: Create mount targets > Delete default SG's (blue bloxes) > Attach APP subnets: us-east-1a, "sn-app-A" etc > Security Group x3 "IMPLEMENTINGEFS[...]" > Next > Skip File System policy (optional) > Next > Review and CREATE
NOTE: any AZs within a VPC you're consuming the services provided by EFS, you should be creating a mount target
4. EC2 Instances > duplicate EC2 instance tab > tab 1, EC2 connect to instance A, tab 2 connect to B
5. EC2 Connect Instance A > `sudo mkdir -p /efs/wp-content` (creates file as well as any paths like /efs/ with the -p) >
`sudo dnf -y install amazon-efs-utils` package of tools which allows this instance to interact with EFS >
`cd /etc` >
`sudo nano /etc/fstab` >
Paste this into 3rd line of nano: `file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0` >
6. Paste EFS file system ID of A4L EFS into the nano text above, replacing 'file-system-id`
7. Save Nano: ctrl+o to save, Enter, ctrl+x to exit
8. `df -k` still not showing anything. Time to mount file system: `sudo mount /efs/wp-content`, now `df -k` will show it >
`cd /efs/wp-content` (move into new file system) >
`sudo touch amazingtestfile.txt` (create test file)
9. Instance B Connect:
df -k >
sudo dnf -y install amazon-efs-utils >
sudo mkdir -p /efs/wp-content >
sudo nano /etc/fstab >
file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0 >
sudo mount /efs/wp-content >
cd /efs/wp-content/ >
ls -la -- you'll see amazingtestfile.txt in the directory which was created in instance A
10. Clean up: EFS > Delete file system > Cloudformation > Delete stack

## AWS Backup
Use AWS Backup to centralize and automate data protection across AWS services and hybrid workloads. AWS Backup offers a cost-effective, fully managed, policy-based service that further simplifies data protection at scale. AWS Backup also helps you support your regulatory compliance or business policies for data protection. Together with AWS Organizations, you can use AWS Backup to centrally deploy data protection policies to configure, manage, and govern your backup activity across your company’s AWS accounts and resources.
- fully managed data-protection
- consolidate backups across accounts/regions into one place

### AWS Backup - Components
- Backup Plans: frequency, window, lifecycle, vault, region copy
- Backup resources: the resources being backed up
- Vaults: destination for backups (container). Assign KMS key for encryption
- Vault Lock: WORM write-one-read-many, 72 hour cool off, then even AWS can't delete
- On-Demand backups when you require
- PITR: Point in time recovery


# HA & Scaling

## Regional and Global AWS Architecture
Global Components - Global Svc Location/Discovery, Content Delivery+optimization, global health checks/failover
Regional Components - Regional entry point, scaling/resilience, app svc's/components

## Evolution of Elastic Load Balancer (ELB)
Currently 3 types of ELBs available in AWS split between v1 (avoid this) and v2 (use this). "ELB" refers to all 3.
- Version 1: Classic Load Balancer (CLB), 2009. Not really layer 7, only one SSL per CLB
- Version 2:
-- v2-1. Application Load Balancer (ALB): HTTP/HTTPS/WebSocket. Layer 7 device
-- v2-2. Network Load Balancer (NLB): TCP, TLS, UDP. For apps that don't use HTTP/HTTPS, like email or SSH servers
- Version 2 is faster, cheaper, supports target groups and rules

## Elastic Load Balancer (ELB)
Job of a Load Balancer is to accept connections from a user and distribute that connection to the backend of the app
- "dual stack" means using both IPv4/6
- config'd to run in 2+ AZ's. 1+ nodes are placed into a subnet in each AZ and scale with load
- Each ELB confg'd with an A record DNS name which resolves to the ELB nodes. Requests distributed to nodes, nodes scale within AZ
EXAM: ELB can be internet-facing (given public and private IP Addresses) or internal (given only private IP addresses)
EXAM: Internet-Facing LB can still access both public AND private EC2 instances
EXAM: /27 or larger subnet is minimum for load balancer. /28 is 2nd most correct answer

### ELB - Cross-Zone Load Balancing
- Load Balancer Nodes can send requests across an AZ to any other registered instance in other AZs for further balancing
- Comes enabled as default

## ELB EXAM
EXAM: ELB is a DNS A Record pointing at 1+ Nodes per AZ
EXAM: Nodes can scale per subnet
EXAM: Internet-facing nodes have public IPv4 IPs
EXAM: Internal-facing nodes have only Private IPs
EXAM: EC2 instance does NOT need to be public to work with an internet-facing LB
EXAM: Listener Config controls WHAT the LB does
EXAM: ELBs required 8+ free IPs per subnet, and at least /27 subnet to allow scaling


## Application Load balancing (ALB) vs Network Load Balancing (NLB)
When to pick ALB vs NLB.
- Remember, always avoid Version 1 Classic Load Balancer -- does not scale.

### Version 2, APPLICATION Load Balancers:
- Layer 7 LB's. Listens on either HTTP and/or HTTPS
- Doesn't listen to any other Layer 7 protocols (SMTP, SSH, Gaming, etc)
- Can't listen to TCP/TLS/UDP
- Can't do end-to-end unbroken SSL encryption as the request is terminated on ALB and new connection is made from ALB -> App
NOTE: if you need unbroken encryption, you need to use a Network Load Balancer
- ALBs using HTTPS must have an SSL cert on that LB
- ALBs are slower than NLB as they're more complex
- As they are Layer 7, they can do health checks at Layer 7
- "Rules" - Direct connections which arrive at a listener, rules processed in priority order, there is a final Default Rule as a catch-all
- "Rule Conditions"
- "Actions" - follows rules
NOTE: if you need unbroken encryption, you need to use a Network Load Balancer

### Version 2: NETWORK Load Balancers
- Layer 4 LBs: TCP, TLS, UDP, TCP_UDP
- No visibility or understanding of HTTP/S; can't see headers, cookies, session stickiness
- NLBs are super fast, about 25% of ALB latency
- Good for SMTP, SSH, game servers which don't use web protocols, financial apps which don't use web protocols
- health checks not app aware, they just check ICMP/TCP handshakes
- can have static IPs which is useful for whitelisting
- can forward TCP to instances for UNBROKEN ENCRYPTION
- used for PRIVATE LINK to provide services to other VPCs

EXAM: Need unbroken encryption? Network Load Balancer
EXAM: Need highest performance LB? NLB
EXAM: Need Private Link? NLB
EXAM: Static IP? NLB

EXAM--
### ALB VS NLB
- Need unbroken encryption? Network Load Balancer
- Static IP for whitelisting? NLB
- Best performance? NLB
- Operate on non-HTTP/S? NLB
- Private Link? NLB
- Otherwise... ALB

## EC2 Launch Configuration and Templates
Both alllow you to define config of an EC2 instance in advance
- Including....
-- AMI, instance type, storage & keypair, networking and SGs, userdata and IAM role
- Configs are NOT editable; defined one. But, Launch Templates have versions
- LT came after LC so it has newr features like T2/T3 unlimited, placement ggroups, capacity reserveations, elastic graphics
NOTE: AWS recommends using Launch Templates over LCs
- LC's have one use: only used as part of Auto Scaling groups

## Auto Scaling Groups
Auto Scaling group contains a collection of Amazon EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management. An Auto Scaling group also enables you to use Amazon EC2 Auto Scaling features such as health check replacements and scaling policies. Both maintaining the number of instances in an Auto Scaling group and automatic scaling are the core functionality of the Amazon EC2 Auto Scaling service.
- "seal healing" for EC2
- uses launch templates or configs to launch all scaled instances
- Auto-scaling group has three important values: Minimum size, desired capacity, maximum size (eg 1:2:4), called min/desired/max or x/y/z
- Auto-scaling keeps running instances at the DESIRED capacity by provisioning/terminating instances
- Scaling Policies automate based on metrics like CPU load

### Auto-Scaling Groups - Scaling Policies
Scaling policies are rules. Three ways you can scale auto-scaling groups:
1. Manual Scaling. Manually adjust min/desired/max
2. Scheduled Scaling. Time-based adjustment
3. Dynamic Scaling (3 subtypes). Rules which react to something and change
- a. Simple Scaling. Often a pair of rules: "CPU above 50%,  +1. CPU below 50%, -1."
- b. Stepped Scaling. Bigger +/- based on difference (generally use this over Simple unless simplicity is the priority)
- c. Target Tracking. Eg.. Desired aggregate CPU to stay at 40%
Cooldown Periods: How long to wait at the end of a scaling action before doing another

### ASG - Health
ASG performs healths checks using EC2 status checks
- Self healing: If ASG detects downed instance, it will replace the downed instance with a newly provisioned one

### ASG - ASG + Load Balancers
- Uses Target Groups
- Can use Load Balancer health checks instead of EC2 health checks; application aware health checks (which ec2 health checks are not)

### ASG - Scaling Processes
- LAUNCH and TERMINATE (can SUSPEND / RESUME) any process, like suspending the Launch process to not scale out if event takes place
- AddToLoadBalancer - Add to LB on launch
- AlarmNotification - Accept notification from CloudWatch
- AZRebalance - Balances instances evenly across all AZs
- HealthCheck - health checks on/off
- ReplaceUnhealthy
- ScheduledActions
- Standby or InService

### ASG - Final Points
EXAM: Autoscaling groups are free; only the resources created are billed (use cooldowns to avoid rapid scaling and increased costs)
EXAM: Think about using more smaller instances for granularity - higher control and more cost-effective
EXAM: Use with Application Load Balancers for elasticity; abstraction
EXAM: ASG defines WHEN/WHERE, Launch Templates/Configs define WHAT

## ASG - Scaling Policies
Scaling policies are NOT REQUIRED on ASGs
- Manual - When you set Min / Desired / Max. Best for Testing or when urgent
- Step / Simple scaling: you choose scaling metrics and threshold values for the CloudWatch alarms that trigger the scaling process. You also define how your Auto Scaling group should be scaled when a threshold is in breach for a specified number of evaluation periods
-- Step / Simple scaling require you to create CloudWatch alarms for the scaling policies. Both require you to specify the high and low thresholds for the alarms
-- Step / Simple scaling require you to define whether to add or remove instances, and how many, or set the group to an exact size
- Scaling based on SQS - ApproximateNumerOfMessagesVisible

## ASG - Lifecycle Hooks
Lifecycle hooks enable you to perform custom actions by pausing instances as an Auto Scaling group launches or terminates them. When an instance is paused, it remains in a WAIT state either until you 1. complete the lifecycle action using the complete-lifecycle-action command or 2. the CompleteLifecycleAction operation, or 3. until the timeout period ends (one hour by default) then CONTINUE or ABANDON.
- Configure Custom Actions on instances during ASG actions (launch or terminate transitions)
- Can be config'd with EventBridge or SNS Notifications

## ASG - Health Check Comparison: EC2 VS ELB
Amazon EC2 Auto Scaling can determine the health status of an instance using one or more of the following:
- Amazon EC2. To identify hardware and software issues that may impair an instance. This is the Default for ASG
- Elastic Load Balancing (ELB). Disabled by default. ELB health checks can be application aware (Layer 7)
- Custom Health Checks. Instances marked Healthy/Unhealthy by an external system

EXAM: Health Check Grace Period (default 300s); a delay before starting health checks which first allows system to launch, bootstrap, application start

## Elastic Load Balancer - SSL Offload & Session Stickiness
Three ways a load balancer handles secure connections: Bridge, Pass Through, Offloading
- Bridging (default). SSL encrypt/decrypt happens at ELB, but instances still need SSL certs and compute required for crpyto operations
- Pass Through. No encypt/decrpyt happens at ELB, instead each instance has SSL Cert installed
- Offload. SSL decrypted at ELB and then only plain text HTTP goes through to instance. Instance doesn't need SSL cert or crypto compute capability

### ELB - Connection Stickiness
- With no stickiness, connections are distributed across all in-service backend instances
- With stickiness, a cookie is generated (AWSALB) which locks the device to a single backend instance for a set duration: 1 second to 7 days
NOTE: Instead of worrying about this, just make sure servers are stateless and state is stored elsewhere like the DB

## DEMO - EMB - Seeing Session Stickiness in Action
How session stickiness works with Application Load Balancers
Video: https://learn.cantrill.io/courses/1820301/lectures/43242687

## ADVANCED DEMO - Architecture Evolution
Stage 1 - Setup the environment and manually build wordpress - https://learn.cantrill.io/courses/1820301/lectures/41301448
Stage 2 - Automate the build using a Launch Template
Stage 3 - Split out the DB into RDS and Update the LT
Stage 4 - Split out the WP filesystem into EFS and Update the LT
Stage 5 - Enable elasticity via a ASG & ALB and fix wordpress (hardcoded WPHOME)
Stage 6 - Cleanup

## Gateway Load Balancer (GWLB)
Gateway Load Balancers enable you to deploy, scale, and manage virtual appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems. It combines a transparent network gateway (that is, a single entry and exit point for all traffic) and distributes traffic while scaling your virtual appliances with the demand.
- At high level, GWLBs have two main components:
1. Endpoints. Traffic enters/leaves via these endpoints
2. The GWLB itself; balances packets across multiple backend instances

- Traffic and metadata is tunneled using GENEVE protocol
- GWLB = network security at scale


## Serverless and Application Services

### Architecture Deep Dive - Event-Driven Architecture
Types of architecture:
- MONOLOTHIC: everything is built together and coupled; everything is always running and incurring charges even if not being used; if one part fails, whole thing fails
- TIERED: monolith broken apart into tiers that can be on same server or different servers. Tiers still coupled as they each connect to endpoint of next tier. Benefits are that tiers can be vertically scaled independently (server size of each tier can be increased).
-- if you add load balancers between tiers, this abstracts tiers meaning that you can now horizontally scale tiers by adding instances (the tiers are no longer connected to each other but to LBs. Still not decoupled though.
- QUEUE-BASED Decoupled Architecture: The queue would sit between components (tiers) to decouple, the components no longer require answers; now the components are using async communications
- MICROSERVICE Architecture: tiers/components become Microservices that can be duplicated. There are Producers, Consumers, and Both
-- Producers produce data, Consumers consume data/messages, 'Both' does ..both
--- Events are what are consumed. Queues can be used to communicate events
- EVENT DRIVEN Architecture: Event Producers / Event Consumers. You can have components that can do Both
-- Best-practice Event-driven arch's have Event Routers, which are highly available central exchange points for Events. Nothing is sitting around waiting, they are activated when an Event is sent to them; only consumes resources while handling events (serverless)

### Lambda
- FaaS: Function as a Service - short running and focused
- Lambda function: piece of code that lambda runs
- Functions use a runtime like Python 3.8
- Functions loaded into a runtime environment; env has a direct memory (indirect CPU) allocation
- Billing: for the duration that the function runs
- Lambda is a key part of Serverless architectures
- Lambda Function: Think of it as the code + association wrappings/config; a deploment package. Stateless
- You define Lambda resources like Memory (128MB -> 10240MB in 1MB steps), which then scales how much CPU you'll get (you can't directly control how much CPU). Storage in Lambda stored in /tmp 512MB -> 10240MB

EXAM: Lambda function timeout 900s (15 minutes). Anything beyond 15 mins can't use Lambda directly

#### Common Uses of Lambda
- Serverless applications
- File processing
- Database triggers
- Serverless CRON
- Real time stream data processing (kinesis + lambda)

#### Lambda Networking Modes
1. Public (Default)
2. VPC

1. Public: Could access Public AWS svc's like SQS, DynamoDB, or external like IMDB
- Lambda functions have no access to VPC unless public IPs are provided and security controls allow external access
2. Private (in VPC): Lambda functions running in a VPC obey ALL VPC networking rules (can't access external stuff unless configured as such). Eg. Could use a gateway endpoint to access dynamodb

#### Lambda - Security
- Execution Role: For Lambda env to access AWS p&s it needs an Execution Role to gain the role's permissions based on the role policy
- Lambda Resource Policies: similar to bucket policy on S3 which controls WHO/WHAT can interact with a specific Lambda function. Eg. services like SNS or S3 being allowed to invoke the function

### Lambda - Logging
Lambda uses CloudWatch, CloudWatch Logs, and X-Ray
- Logs from Lambda executions -> CloudWatch Logs
- Metrics like invocation success/failures, retries, latency -> CloudWatch
- Distributed Tracing Capability -> X-Ray. Eg. Trace the path of a user through an application
NOTE: CloudWatch Logs requires permissions via the Execution Role

### Lambda - Invocation
Types:
- Synchronous
- Asynchronous
- Event Source Mappings

Synchronous: CLI/API invokes lambda function and then CLI/API WAITS for a response. Errors/Retries/Reprocessing must happen on the client side. Synchronous is generally invoked by a human

Asynchronous: Typically used when AWS svc's invoke lambda functions. Reprocessing is handled by Lambda
- Lambda function needs to be idempotent; reprocessing a result should have the same end-state. Eg. Adding +$10 to an account or Setting account to $10 explicitly. If adding fails in a certain way, you could end up with 20, 30, 40 etc. Idempotent failures still explicitly set the account to $10 value over and over
- Events can be sent to DLQ's (dead letter queues) after repeated failed processing
- Lambda support destinatoins where successful or failed events can be sent like SQS, SNS, Lambda & EventBridge

Event Source Mapping: Typically used on streams or queues which don't support event generation to invoke Lambda (Kinesis data stream, DynamoDB streams, SQS queues, Amazon Managed streaming for Apache Kafka). Event Source Mapping polls these streams/queues looking for data and getting back 'source batches', split based on batch size, sent to Lambda function

### Lambda - Versions
Lambda can have versions... v1, v2, v3. The VERSION of Lambda function is 'code + config'
- Versions are immutable when published and has its own ARN. $Latest points at the ..latest... version
- Lambda functions can have aliases like DEV, STAGE, PROD which can be changed

### Lambda - Startup times
- An 'execution context' is the environment a lambda function runs in
- A 'cold start' is a full creation and configuration including function download
- A 'warm start' can occur if another lambda function is invoked shortly after a first, now it doesn't have to rebuild the execution context (no build process)
- Event source mapping uses permissions from the Execution Role to access the source svc that it pulls data from


## CloudWatch Events and EventBridge
CloudWatch Events and EventBridge have visibility over events generated by supported AWS services within an account.
- They can monitor the default account event bus - and pattern match events flowing through and deliver these events to multiple targets.
- They are also the source of scheduled events which can perform certain actions at certain times of day, days of the week, or multiple combinations of both - using the Unix CRON time expression format.

EventBridge is replacing CW Events as EventBridge can do all the same things but also handle third-party and custom app events (AWS recommending using EventBridge)

### Key Concepts, CloudWatch Events and EventBridge
- If X happens, or at Y times... do Z
- EventBridge is basically CW Events v2
- A default Event Bus exists on an AWS account for both CW Events and EB
- In CW Events, only 1 event bus. In EB you can have add't event busses
- Rules created match incoming events (or schedules)
-- Two rule types: 1) Event Pattern Rule 2) Schedule Rule


## DEMO - Automated EC2 Control Using Lambda and Events
Gain experience of using Lambda for some simple account management tasks.

1. 1 click deploy
2. Give Lambda Permissions: https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0024-aws-associate-lambda-eventdrivenlambda/lambdarole.json > IAM > Role >  Create Role > Entity Type, AWS Service, select Lambda > Next
3. Create IAM Role Policy: Click 'Create Policy' > JSON tab > Paste JSON provided > Next > Policy name "Lambdastartandstop" > Create Policy
4. Create IAM Role with Policy: IAM Role tab > Select Policy for IAM Role > Role name "EC2StartStopLambdaRole" > Create Role
5. EC2 > Copy both instance ID's to notepad/clipboard
6. Create Lambda Function: Lambda > Create function > select Author from Scratch > name "EC2Stop" > runtime "Python 3.9" > dropdown "Change default exec role", select Use Existing, select the IAM Role you created > Create Function > paste in code to Lambda code block from file lambda_instance_stop.py > click Deploy
7. Creat env variable for EC2Stop: Lambda function, configuration tab > Environment Variables "Edit" > click "Add env variable" > key: EC2_INSTANCES, value: "ec2_instance_ID_1,ec2_instance_ID_2"
8. Test Event: EC2Stop Lambda Function, Test tab > Event name "test" > Test. Observe EC2 instances being stopped.
9. Create second function "EC2Start" > Lambda, Functions "Create Function" > name "EC2Start" > Existing IAM Role > Create function > add new code from lambda_instance_start.py > Paste code, Deploy >  Environment Variables "Edit" > click "Add env variable" > key: EC2_INSTANCES, value: "ec2_instance_ID_1,ec2_instance_ID_2" > Save
10. Test Event: EC2Start Lambda Function, Test tab > Event name "test" > Test. Observe EC2 instances being started.
11. Set up Event-driven Lambda Function: Lambda > New Function name "EC2Protect" > Runtime python 3.9 > Select IAM role that we created > Create function > Add code from file "lambda_instance_protect.py", Deploy
12. Create EventBridge Rule for Lambda to receive: EventBridge > New Rule name "EC2Protect" > desc "Start protected instance" > select "rule with an event pattern" > Event Source "AWS events or EventBridge partner events" > Event pattern "AWS Services", "EC2", event type: EC2 Instance State-change Notification, Specific States: Stopped > Specific Instance IDs, paste instance 1 ID > Next > Target 1 AWS Svc, "Lambda function", Function: EC2Protect > Create Rule
13. Clean up: Lambda, delete functions > EventBridge, delete Rules > IAM Policies, delete Lambda policy > Delete Lambda IAM Role > CloudFormation, delete Stack

## Serverless Architecture
The Serverless architecture is a evolution/combination of other popular architectures such as event-driven and microservices.
- Aims to use 3rd party services where possible and FAAS products for any on-demand computing needs.
- Using a serverless architecture means little to no base costs for an environment - and any cost incurred during operations scale in a way with matches the incoming load

### Serverless - Key Concepts
- Serverless isn't one single thing; aiming to manage few, if any, servers for lower overhead
- Stateless and Ephemeral env's; duration billing
- Function as a Service; FaaS used where possible for Compute functionality
- Managed Services are used where possible; web Id providers, S3 for storage, etc.
- Event-driven; consumption only when being used

### Serverless: TL;DR and Architecture
Aim should be to consume as a service whatever you can, code as little as possible, use function as a service for any general compute needs

## SNS - Simple Notification Service
The Simple Notification Service or SNS is a PUB / SUB style notification system which is used within AWS products and services but can also form an essential part of serverless, event-driven and traditional application architectures.
- Public AWS Service; network connectivity with a Public Endpoint
- Publishers send messages to TOPICS
- Subscribers receive messages SENT to TOPICS
- Messages are <= 256KB payloads
- SNS supports a wide variety of subscriber types including other AWS services such as LAMBDA and SQS
- SNS is REGIONALLY RESILIENT. Highly available and scalable
- Can provide delivery STATUS and delivery RETRIES
- SNS capable of server-side encryption
- SNS can be cross-account with a TOPIC POLICY

## AWS Step Functions
AWS Step Functions addresses some problems within Lambda.
- Lambda is FaaS, each Lambda should be as simple as possible; you'd never put a full application in a Lambda Function (execution duration limit: 15 minutes). Theoretically, you can chain Lambda functions but this gets messy at scale
- Lambda runtime env's are stateless (can't hold state through different Lambda functions)

Step Functions let you create STATE MACHINES (a workflow with start point and end point). START -> STATES -> END
- States are THINGS which occur
- Max Duration of State Machines is 1 YEAR
- Two workflows in State Machines:
1. Standard Workflow (default); 1 YEAR execution limit
2. Express Workflow: designed for high-volume, event-processing workloads, etc for UP TO 5 MINUTES
- You can Export your State Machine as a JSON template called "Amazon States Language (ASL)"
- IAM Role used for permissions

### Step Functions: State Machines: Types of States
- Succeed and Fail.
- Wait. Wait for certain period of time to elapse, or for a particular date/time that was set. Holds / Pauses workflow until duration passes
- Choice. State Machine can take different paths depending on input
- Parallel. Create parallel branches within a state machine
- Map. Takes list of items and performs action/set of actions on each item in list
- Task. Represents a Single unit of work performed by a State Machine

## API Gateway 101
API Gateway is a managed service from AWS which allows the creation and management of API Endpoints, Resources & Methods. Endpoint/entry-point for applications
- The API gateway integrates with other AWS services - and can even access some without the need for dedicated compute.
- It serves as a core component of many serverless architectures using Lambda as event-driven and on-demand backing for methods.
- It can connect to legacy monolithic applications and act as a stable API endpoint during an evolution from a monolith to microservices and potentially through to serverless.
- API Gateway sites between applications and integrations (services)
- Can connect to svc's/endpoints in AWS or ON-PREMISES
- HTTP APIs / REST APIs / WEB SOCKET APIs

### API Gateway - Authentication
- Can use COGNITO user pools for authentication
- Can use LAMBDA for authentication

### API Gateway - Endpoint Types
- Edge-optimized. Incoming requests routed to nearest CloudFront POP (point of presence)
- Regional. Clients in the same region
- Private. Endpoint accessible only within a VPC via interface endpoint

### API Gateway - Stages
- When you deploy an API config in API gateway, you do so in a stage. APIs are deployed to stages, each stage has one deployment. Eg. prod and dev stages

### API Gateway - ERRORS
EXAM - Memorize numbers
- Error codes generated by API gateway 2 categories:
1. 4xx - 400 series client-side error
2. 5xx - 500 series server-side error
- Error Codes:
-- 400: Bad request; generic; many root causes possible
-- 403: Access Denied; authorizer denies or WAF filtered
-- 429: Throttling is occuring in API Gateway
-- 502: Bad gateway excption; bad output returned by Lambda
-- 503: Service unavailable. Backing endpoint offline or major service issues
-- 504: Integration Failure/Timeout - 29s limit for any requests to API gateway
Resource: https://docs.aws.amazon.com/apigateway/latest/api/CommonErrors.html

### API Gateway - Caching
- Caching is configured per Stage
- Cache TTL default: 300 seconds; configurable min/max 0s/3600s
- Can be encrypted
- Cache size: 500MB -> 237GB

## DEMO - Build a Serverless App - Skipping for now

## Simple Queue Service (SQS)
SQS queues are a managed message queue service in AWS which help to decouple application components, allow Asynchronous messaging or the implementation of worker pools.
- SQS comes in 2 types:
1) Standard - best efforts; messages could be received out of order
2) FIFO - guarantees the order
- Like SNS, messages <= 256KB in size
- Polling: When a client checks for messages on a queue
EXAM -- Received messages are HIDDEN (VisibilityTimeout - amount of time a msg is hidden). If the message isn't explicitly deleted, it will reappear (retry) in the queue
- Dead-Letter Queue: Can be used for problem messages. Eg if message is received 5 or more times
- SQS good for scaling: ASG's can scale based off SQS, Lambdas can invoke based on queue length -> This is good for ASG Worker Pool Architecture
EXAM - Fanout Architecture: SNS fans messages out to multiple SQS queues

### Delivery Methods - Standard / FIFO
- Standard = guaranteed to deliver 'at-least-once' with no promise on order of delivery
- FIFO = guaranteed to deliver 'exactly-once' and guarantee message order
-- FIFO performance is lower: 3,000 messages / second with batching or 300 messages/second without batching. Not as good for scaling as Standard
- Billing: Based on "requests". A request can be 1-10 messages up to 64KB total
- Polling Types (2)
1) Short Polling (immediate)
2) Long Polling (waitTimeSeconds, up to 20s) - BEST PRACTICE; more cost effective as less requests per message. Will poll for a 20s duration collecting up to 10 messages and 64KB in data
- SQS supports encryption at rest (KMS) and in-transit
- Can add Queue policies

## SQS -  SQS Standard vs FIFO Queues
### Standard
- Multi-lane highway. Scalable, nearly unlimited Transactions per second
- Best-efforts ordering; no rigid order
- Best for decoupling, worker pools, Batch for future processing
### FIFO
- Single-lane highway (performance limited by width of lanes). 300 TPS w/o batching, 3,000 TPS with
- Delivered in a guaranteed order
EXAM - FIFO queues Must have a .fifo suffix
- Exactly-once processing: removes chance of duplicate message delivery

## SQS - Delay Queues
Delay queues provide an initial period of invisibility for messages; postpone delivery to consumers. Predefined periods can ensure that processing of messages doesn't begin until this period has expired.
- A message is automatically hidden once it hits a queue
- NOT the same as Visibility Timeout; VT is automatic re-processing of problem messages whereas this is an initial delay in processing of all messages
- DelaySeconds min is 0, it has to be non-zero to be a Delay Queue
- NOT SUPPOERTED ON FIFO QUEUES

## SQS - Dead-Letter Queues (DLQ)
Dead letter queues allow for messages which are causing repeated processing errors to be moved into a dead letter queue.
- Eg. Message received but keeps re-appearing after visibility timeout and not explicitly deleted (this could technically happen forever unless you set up a DLQ)
-- Each time a message re-appears in queue after timeout, the ReceiveCount tally is incremented
-- when ReceiveCount > maxReceiveCount && message is NOT deleted, it's moved to DLQ

## Kinesis Data Streams
Kinesis data streams are a real-time, scalable streaming service within AWS designed to ingest large quantities of data and allow access to that data for consumers
- Ideal for dashboards and large scale real time analytics needs
- Kinesis data firehose allows the long term persistent storage of kinesis data onto services like S3
- Producers send data into a kinesis STREAM, which can scale from low to near infinite data rates
-- Streams store a 24-hour moving window of data
--- This storage is included and can be increased to MAX 365 days of moving data window (add'l costs)
--- Multiple consumers access data from this moving window

### Kinesis VS SQS
- SQS = decoupling, asynchronous communications. No message persistence, no window of data stored
- Kinesis = huge-scale data ingestion, real-time data streaming. Made for multiple consumers and has a rolling window of stored data for persistence (default 24 hrs, up to 365 days for add'l cost)

## Kinesis Data Firehose
Fully managed service to load data for data lakes, data stores (like S3), and analytics svc's -> This lets data be persisted beyond the rolling window of Kinesis Data Streams
- Auto scaling, fully serverless, resilient
EXAM - NEAR real-time delivery (~60s) which is unlike the main Kinesis Data Stream which offers real-time delivery
- Firehose supports transformation of data on the fly using Lambda
- Valid destination endpoints for Firehose: HTTP, Splunk, Redshift, ElasticSearch, S3

## Kinesis Data Analytics
Amazon Kinesis Data Analytics is the easiest way to analyze streaming data, gain actionable insights, and respond to your business and customer needs in real time.
- Real-time processing of data using SQL
- Ingests from Kinesis Data Streams or Firehose
- Destinations of analyzed data: Firehose (and subsequently S3, Redshift, ElasticSearch, Splunk), AWS Lambda, Kinesis Data Streams
-- If OUTPUT to firehose, data becomes NEAR REAL-TIME. If output to Kinesis Data Streams or Lambda data stays REAL-TIME

### When to use Kinesis Data Analytics
- streaming data needing real-time SQL processing
- time-series analytics like elections, esports
- real-time dashboards like leaderboard for games
- real-time metrics like for security/response teams

## Kinesis Video Streams
Amazon Kinesis Video Streams makes it easy to securely stream video from connected devices to AWS for analytics, machine learning (ML), playback, and other processing. Kinesis Video Streams automatically provisions and elastically scales all the infrastructure needed to ingest streaming video data from millions of devices
- ingest LIVE video data from things like security cameras, phones, cars, drones, time-serialised audio, thermal, depth, RADAR
- consumers can access data frame-by-frame or as-needed
- Can PERSIST and ENCRYPT (in-transit and at-rest)
EXAM: CANNOT directly access the data via storage, only via APIs
- Integrates with other AWS svc's like REKOGNITION and CONNECT

## Amazon Cognito - User and Identity Pools
A user pool is a user directory in Amazon Cognito. With a user pool, your users can sign in to your web or mobile app through Amazon Cognito
- Users can also sign in through social identity providers like Google, Facebook, Amazon, or Apple, and through SAML identity providers
- Whether your users sign in directly or through a third party, all members of the user pool have a directory profile that you can access through a Software Development Kit (SDK).
- Amazon Cognito identity pools (federated identities) enable you to create unique identities for your users and federate them with identity providers. With an identity pool, you can obtain temporary, limited-privilege AWS credentials to access other AWS services.

Cognito provides the following for web/mobile apps:
- authentication
- authorization
- user management

Cognito has 2 components:
1. User pools: for sign-in. You get a JSON Web Token (JWT) if successful (but most AWS svc's don't use this JWT)
- user directory mgmt/profiles, sign up and sign in (with UI), MFA and other security features
- most AWS services CANNOT be accessed using JWT via User Pools
- can use internal or social logins for sign in
2. Identity pool: Swap external ID for temporary AWS credentials.
- Unauthenticated ID's; guest users

USER POOLS = SIGN IN / SIGN UP
IDENTITY POOLS = ABOUT SWAPPING ID TOKENS (with Google, Apple, etc) for temporary tokens to access AWS services

### Cognito - Web ID Federation (using User Pools and Identity Pools in conjunction)
- First, User Pool JWT replaces all the different external token configs you'd need for handling different external IDs (User Pool removes admin overhead of handling all external social logins)
- Next, app can pass this User Pool token through to an Identity Pool
- Federated Identities can swap (Eg. give Google creds to swap for AWS creds)

## AWS Glue
- Fully managed SERVERLESS extract, transform, and load (ETL) service which makes it easy for customers to prepare and load their data for analytics
-- Can create and run an ETL job with a few clicks in the AWS Management Console.
- Simply point AWS Glue to your data stored on AWS, and AWS Glue discovers your data and stores the associated metadata (e.g. table definition and schema) in the AWS Glue Data Catalog. Once cataloged, your data is immediately searchable, queryable, and available for ETL.
- Moves/transforms data between source/destination
- Data Sources: Stores: S3, RDS, JDBC Compatible, DynamoDB. Data Sources: Streams: Kinesis Data Stream, Apache Kafka
- Data Targets: S3, RDS, JDBC Databases
- Crawls data sources and generates the AWS Glue Data Catalog

### AWS Glue Data Catalog
- Persistent metadata about data sources within a region
- One catalog per region per account
- Configure crawlers for data sources

### Glue Jobs
Extract, transform (using script), and load Jobs
- Can be initiated manually or via EVENTS using EventBridge

### Glue VS Data Pipeline (both do ETL)
- Glue is serverless, ad-hoc, more cost-effective
- Data Pipeline requires a server

## Amazon MQ
AmazonMQ is an open-source message broker; an AWS implementation of Apache ActiveMQ. Like a merge of SNS/SQS
- Supports open standards such as JMS, AMQP, MQTT, OpenWire and STOMP
-- If you need to support any of these, and use QUEUES and TOPICS - AmazonMQ is the tool to use.
- Provides Queues and Topics (like SQS/SNS); one-to-one or one-to-many
EXAM: Amazon MQ is NOT a public service; runs on a VPC and requires private networking to access it
EXAM: AWS Integration required? Use SNS/SQS and not MQ.
EXAM: Migrate from an existing system with little to no app change? Amazon MQ
EXAM: Need APIs like JMS or protocols like AMQP, MQTT, OpenWire, STOMP needed? Amazon MQ

## Amazon AppFlow
Amazon AppFlow is a fully managed integration service that enables you to securely transfer data between Software-as-a-Service (SaaS) applications like Salesforce, SAP, Zendesk, Slack, and ServiceNow, and AWS services like Amazon S3 and Amazon Redshift, in just a few clicks.
- With AppFlow, you can run data flows at enterprise scale at the frequency you choose: on a schedule, in response to a business event, or on demand.
- You can configure data transformation capabilities like filtering and validation to generate rich, ready-to-use data as part of the flow itself, without additional steps.
- AppFlow automatically encrypts data in motion, and allows users to restrict data from flowing over the public Internet for SaaS applications that are integrated with AWS PrivateLink, reducing exposure to security threats.
-- Can use AppFlow Custom Connector SDK to build your own integration if the integration doesn't already exist
- Exchange data between applications (connectors) using FLOWS; SYNC or AGGREGATE data
- PUBLIC endpoints, but works with PrivateLink
- CONNECTIONS store config/creds to access apps. Connections can be reused across many flows


# GLOBAL CONTENT DELIVERY AND OPTIMIZATION
## Cloudfront Architecture
CloudFront is a Content Delivery network (CDN) within AWS.
- Origin: The source location of your content (can be S3 or custom origin)
- Distribution: The 'configuration' unit of CloudFront
- Edge Locations: Local cache of your data
- Regional Edge Cache: Larger version of an Edge Location, provides another layer of caching
NOTE: Can use SSL certs with CloudFront as it integrates with AWS Certificate Manager (ACM) for HTTPS
NOTE: CloudFront is for DOWNLOAD operations only, uploads go directly to Origin with NO WRITE CACHING. CloudFront does Read-Only caching

## CloudFront (CF) - Behaviors
CloudFront Behaviors control much of the TTL, protocol and privacy settings within CloudFront
- Behaviors have caching options, and restrict viewer access options
- Distribution can have multiple behaviors, but there is one Default(*) behavior

## CloudFront - TTL and Invalidations
How CloudFront handles object expiry and invalidation
- Covering

- TTL: Object considered not expired when within its TTL
Default TTL: 24 hours (validity period)
Minimum TTL / Maximum TTL can be set

You can specify TTLs using headers.
-- Origin Header Types:
--- Cache-Control max-age (seconds until expiry)
--- Cache-Control s-maxage (seconds until expiry)
--- Expires (Date & Time for expiry)

- Cache Invalidation: Performed on a distribution and applied to all Edge Locations
-- immediately expires objects regardless of their TTL based on the invalidation pattern that you specify
-- Instead of Cache Invalidation, you can use Versioned File Names (whiskers1_v1.jpg // _v2.jpg // _v3.jpg) which is a better practice

## AWS Certificate Manager
A service which allows the creation, management and renewal of certificates. It allows deployment of certificates onto supported AWS services such as CloudFront and ALB.
- HTTP was/is simple and insecure. HTTPS introduced a layer of encryption to HTTP encrypting the data in-transit. HTTPS also allows for servers to prove their identity with Certificates using SSL/TLS
-- These certificates get signed by a trusted authority AKA Chain of Trust

- ACM can be a Public or Private Certificate Authority (CA)
EXAM - ACM can generate or import certificates. If self-generated, it can auto-renew. If imported, you are responsible for renewal
EXAM - Certificates can only be deployed out to SUPPORTED services (Eg. pretty much just CloudFront and ALBs, specifically NOT EC2)
EXAM - Reminder, cannot use ACM with EC2
EXAM - Certs can't leave the region they are generated/imported into; cert are region specific
EXAM - To use a cert with an ALB in ap-southeast-2, you need a CERT IN ACM in ap-southeast-2
- EXAM -- GLOBAL services like CloudFront operate as though within us-east-1

## Cloudfront and SSL/TLS
- Each CloudFront distribution gets a Default Domain Name (CNAME DNS record) when created
- SSL supported by default as long as you use *.cloudfront.net cert
-- Can have Alternate Domain Names Eg. cdn.catagram.com
--- You need to verify ownership using a matching certificate
- Global service? Cert must be in us-east-1, Eg. CloudFront

When using CloudFront, you actually have TWO SSL connections:
1. Viewer => CloudFront
2. CloudFront => Origin
- Both need valid PUBLIC certificates. Self-signed certs don't work with CloudFront

## CloudFront (CF) - Origin Types & Origin Architecture
CloudFront origins store content distributed via edge locations.
- The features available differ based on using S3 origins vs Custom origins

### Origin Categories
- S3 Buckets
- AWS Media Package Channel Endpoints
- AWS Media Store Container Endpoints
- Everything else (web servers); Custom Origins. Note: An S3 as a static website is treated not as S3 but as a Web Server aka custom origin

## DEMO - CloudFront (CF) - Adding a CDN to a static Website
Implement a CloudFront distribution using an S3 bucket as the origin.

Steps:
1. 1-click deploy. At this point, site in demo is static S3
2. Host site on CloudFront: CloudFront > Create Distro > Origin domain, select s3 bucket "cfands3-top10cats[...]" > Cache key and origin requests, select "CachingOptimized" > Default root object "index.html" > Deploy/Create
Note: You can now access CDN cached files that are cached in Edge Locations. You can Invalidate (billed per Inval., do it less frequently). In this part, we changed merlin.jpg file to new image and played with caching in CloudFront.
3. Add custom Domain name: Didn't have custom Domain made in R53 so had to watch at this point
4. SSL via ACM:

## Securing CF and S3 using OAI
Origin Access Identities (OAI) are a feature where virtual identities can be created, associated with a CloudFront Distribution and deployed to edge locations.
- Access to an s3 bucket can be controlled by using these OAI's - allowing access from an OAI, and using an implicit DENY for everything else.
-- They are generally used to ensure no direct access to S3 objects is allowed when using private CF Distributions.

### Origin-side Security
#### S3 Origin
You can have S3 Origin, or Custom Origin (S3 static website).
OAI's are a type of Identity that are associated with CF Distros to 'become' that OAI to be used in S3 Bucket Policies ; S3 Origin is locked down to only be accessible by this OAI, all else Implicit Denied.
- Explicit Allow on S3 Policy for OAI
- Best practice: Create 1 OAI for 1 CF Distro
#### Custom Origin - Custom Headers or Traditional Firewall
1. Utilize custom headers and Viewer control policy (HTTPS), then a related Origin Control Policy that has a required Custom Header attached. Origin requires the custom header
2. Traditional Security Method: IP Ranges of CloudFront with access to a Firewall

## CloudFront - Private Distribution & Behaviours
### Signed URLs and Signed Cookies
CloudFront can be Public access or Private Access (requiring signed Cookie or signed URL)
- CF Distro is created with ONE behavior for the whole distro, which is public or private
- It'll end up having multiple behaviors, private and public
Old way required a CloudFront KEY created by account root user and account is added as a TRUSTED SIGNER
New way is TRUSTED KEY GROUPS added which can be managed with CloudFront API

### Signed URLs VS Cookies
- SignedURLs provided access to ONE object
- Signed Cooked for access to GROUPS of objects
- SignedURLs if client doesn't support cookies
- Maintain your URL? Signed Cookies

## DEMO - CloudFront (CF) - Using Origin Access Control (OAC) (new version of OAI) - SKIPPING

## Lambda@Edge
Lambda@Edge allows cloudfront to run lambda function at CloudFront edge locations to modify traffic between the viewer and edge location and edge locations and origins.
- Not the full feature-set of Lambda: only Node.js and Python as run times. No VPC based resources. Lambda Layers not supported. Different limits than Lambda.
- You can run the Lambda@Edge functions at 4 points: 1. Viewer request 2. Origin request 3. Origin response 4. Viewer response

### Lambda@Edge Use Cases:
- A/B Testing (Viewer Request)
- Migration between S3 Origins (Origin Request)
- Different Objects delivered based on Device (Origin Request)
- Content by Country (Origin Request)

## AWS Global Accelerator
AWS Global Accelerator is designed to improve global network performance by offering entry point onto the global AWS transit network as close to customers as possible using Anycast IP addresses
- Designed to optimize the flow of data from user to AWS infrastructure
- Reduce number of "hops"
- When to use CloudFront VS Global Accelerator?
-- CloudFront is focused on improving content delivery to end-users via Edge Caching. Question mentions caching? Prolly CloudFront
-- Global Accelerator is focused on optimizing traffic routing to your applications -- by entering the network closer to the customer. Question mentions TCP/UDP? Prolly Global Accelerator
Global Accelerator is a good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP, as well as for HTTP use cases that specifically require static IP addresses or deterministic, fast regional failover.

### Global Accelerator Resource: https://aws.amazon.com/global-accelerator/faqs/


# ADVANCED VPC Networking
## VPC Flow Logs
VPC Flow logs is a feature allowing the monitoring of traffic flow to and from interfaces within a VPC
- VPC Flow logs can be added at a VPC, Subnet or Interface level.
- Flow Logs DON'T monitor packet contents ... that requires a packet sniffer.
- Flow Logs can be stored on S3 or CloudWatch Logs
NOTE: VPC flow logs ONLY CAPTURE METADATA, NOT CONTENTS

Flow Logs can monitor at 3 levels: 1. VPC 2. Subnet 3. ENIs directly. And these logs capture from capture point and down (so if VPC flow log, it also captures Subnet and ENIs)
EXAM:  - Flow Logs NOT real time
- Log Destinations: S3 or CloudWatch Logs
-- Athena can query the logs in S3
- Flow Logs can capture ACCEPTED, REJECTED, or ALL METADATA

## Egress-Only Internet gateway
Egress-Only internet gateways allow OUTBOUND (and response) only access to the public AWS services and Public Internet for IPv6 enabled instances or other VPC based services
- TL;DR Egress-Only is OUTBOUND-ONLY for IPv6

## VPC Endpoints - Gateway
- Gateway endpoints are a type of VPC endpoint which allow private access to public svc's S3 and DynamoDB WITHOUT using PUBLIC addressing. (normally S3 and DynamoDB are public)
- Gateway endpoints add 'prefix lists' to route table, allowing the VPC router to direct traffic flow to the public services via the gateway endpoint.
-- Gateway Endpoint => 1 per service, per region
-- REGION RESILIENT: Highly Available across all AZs in a region by default
- Endpoint Policy can control what it can access (like certain S3 buckets)
- CANNOT access cross-region services

### VPC Endpoints uses
- Private VPC that needs private access to S3/DynamoDB
- Preventing Leaky Buckets: S3 buckets can be set to private only by allowing access ONLY from a gateway endpoint

## VPC Endpoints - Interface
Interface endpoints are used to allow private IP addressing to access public AWS services.
- DynamoDB is handled by gateway endpoints - other supported services are handled by interface endpoints. S3 now supported by interface endpoints
- Added to specific subnets, an ENI. Not Highly Available
-- For HA, you need an endpoint in each subnet in each AZ used in VPC
EXAM - TCP and IPv4 ONLY
- uses PrivateLink: allows AWS or 3rd party svc's to be injected into your vpc and be given network interfaces
- Apps can access Interface Endpoints via Regional DNS, Zonal DNS, or Private DNS (that overrives default DNS)
-- Private DNS associates a private R53 hosted zone to the VPC changing the default svc DNS to resolve to the interface endpoint IP

## DEMO - VPC Endpoints - Interface - PART1 / PART2 / PART3 - Skipping for now

## VPC Peering
VPC peering is a software defined and logical networking connection between two [AND ONLY TWO] VPC's
- VPCs in the same or different accounts and the same or different regions.
EXAM - TWO VPCs connected only
- If VPCs in same region, SGs can reference peer SGs
EXAM - VPC Peering does NOT support transitive peering (AKA A -> B peering, then B -> C peering... C is not auto peered to A)
- With peering, you're basically setting up gateways in each VPC which requires Routing Configuration on EACH side. SGs/NACLs must be set up to allow traffic through
EXAM - 4 VPCs... how many peering connection to connect all? 6
EXAM - IP ranges of VPC Peers CANNOT OVERLAP

## DEMO - VPC Peering - Skipping for now


# HYBRID ENVIRONMENTS AND MIGRATION
## Border Gateway Protocol 101
Introduction to the Border Gateway Protocol (BGP) which is a routing protocol used by some AWS services such as Direct Connect and Dynamic Site to Site VPNs.
- BGP made of AS (autonomous systems). Routers controlls by one entity; a network in BGP. Your whole company's network system is a 'black box' abstraction, BGP only cares about ingress/egress
-- Autonomous System Numbers (ASNs) are unique and allocated by IANA (0-65535), 64512-65534 are PRIVATE
- BGP operates over tcp/179 -- reliable
- BGP not automatic, peering is manually configured
- BGP is a PATH-VECTOR protocol; it exchanges the best path to a destination between peers...best path (AKA shortest path) is called ASPATH
- iBGP (internal routing within an AS) - most common, eBGP (external routing between ASs)
- ASPATH prepending: A way to make a shorter but slower path look worse than a longer but faster path (by just addking more ASs to the path route)

## IPSec VPN Fundamentals
IPsec VPN negotiation occurs in two phases:
1. In Phase 1, participants establish a secure channel in which to negotiate the IPsec security association (SA)
2. In Phase 2, participants negotiate the IPsec SA for authenticating traffic that will flow through the tunnel and use IPSec Keys to exchange 'interesting traffic'
- sets up secure tunnels across insecure networks
- provides authentication and encryption; a secure connection over an insecure network
- asymmetric encryption used to establish symmetric encryption

### IPSec - Two Phases
IKE Phase 1: Internet Key Exchange. IKE v1 and IKE v2 (v2 is newer)
- authentication with asummetric encryption to agree on and create a shared symmetric key
- "Diffie-Helman Private Key"/DH Key created by both sides in Phase 1.. this is what actually creates the symmetrical key for phase 2
IKE Phase 2: Getting VPN up and running
- built on phase one using the symmetric key created in phase 1
- Phase 2 can be town down and rebuilt when needed, phase 1 can stay

### IPSec - VPN Types (2)
1. Policy Based. Rule sets match traffic. More difficult to configure, but more flexible than Route Based
2. Route Based. Target matching based on prefix
- difference being how they match 'interesting traffic'

## AWS Site-to-Site VPN. AWS <-> on-premises VPN
AWS Site-to-Site VPN is a hardware VPN solution which creates a highly available IPSEC VPN between an AWS VPN and external network such as on-premises traditional networks.
- Quickest way to create network link between AWS and non-AWS (less than an hour)
- VPNs are quick to setup vs direct connect, don't offer the same high performance, but do encrypt data in transit.
- Runs over the public internet (unlessw otherwise specificed)
EXAM - Highly Available

### AWS Site-to-Site VPN - Components
- VPC. VPC connected to external network via VPN
- Virtual Private Gateway (VGW). The target one or more route tables
- Customer Gateway (CGW): Logical piece of config and the thing that the config represents
- VPN Connection

### Static VS Dynamic VPN
- Static uses BGP, Border Gateway Protocol (customer router must support this). Static networking config, static routes
- Dynamic VPN: Need Direct-Connect? Dynamic. Multiple VPN connections for Higher Avail and traffic distribution
- "Route Propagation": if enabled, means routes are added to RTs automatically

EXAM - AWS speed limitation of VPNs 1.25Gbps
EXAM - Latency is inconsistent as it is through Public Internet. If need low latency, maybe Direct Connect

## Direct Connect (DX) Concepts
AWS Direct Connect links your internal network to an AWS Direct Connect location over a standard Ethernet fiber-optic cable. One end of the cable is connected to your router, the other to an AWS Direct Connect router. With this connection, you can create virtual interfaces directly to public AWS services (for example, to Amazon S3) or to Amazon VPC, bypassing internet service providers in your network path.

An AWS Direct Connect location provides access to AWS in the Region with which it is associated. You can use a single connection in a public Region or AWS GovCloud (US) to access public AWS services in all other public Regions.

Direct connect = physical connection. Between business premises <-> DX Location <-> AWS Region
- 1, 10, or 100GBps
- when you order DC, you're really ordering a Port Allocation at the DX Location
- Cannot access internet
- DX Location is NOT owned by AWS (just has AWS space/equipment in it - called a "cage". And you'll have customer or comms partner Cage -- need to connect AWS / Customer router)
- Low latency, high speeds. No resilience (at it's literally 1 cable)
- requires VIFs (virtual interfaces) for the networking over DX to AWS

## Direct Connect (DX) Resilience
Initially, DX has NO resilience as it only has 1 cable. For resilience, you need to lay multiple DX connections / provision multiple ports. Lots of single points of failure by default.
- Best practice... multiple DX Connect Locations... Multiple Customer Premises. Two ports in each DX location, dual routers at customer locations (2)
- At DX Connect Location... Multiple AWS routers, multiple customer routers

## Direct Connect (DX) - Public VIF + VPN (Encryption)
How to use these things to achieve end-to-end encrypted access to private VPC networks across Direct Connect
- Neither public or private VIFS offer any form of encryption.
- Public VIFs+IPSec VPN is a way to provide access to private VPC resources, using an encrypted IPSEC tunnel for transit.

## Transit Gateway
The AWS Transit gateway is a network gateway which can be used to significantly simplify networking between VPC's, VPN and Direct Connect.
- It can be used to peer VPCs in the same account, different account, same or different region and SUPPORTS TRANSITIVE ROUTING between networks.
- Single network object that is highly available and scalable
- create "attachements" to other network types: valid attachments -> VPC, site-to-site VPN, DX Gateway
- Can use "peering attachments" to ve cross-region/cross-account (cross acct = AWS Ram)
- Supports transitive routing (which is basically why it exists, since VPCs don't which causes too much complexity)
EXAM - REMEMBER: TRANSIT gateway supports TRANSITIVE ROUTING (which VPC peering can't do) -- this simplifies vpc-to-vpc

## Storage Gateways - Volume, Tape, File
## Storage Gateway - Volume
Storage gateway is a product which integrates local infrastructure and AWS storage such as S3, EBS Snapshots and Glacier.

### Storage Gateway - Volume Stored Mode
EXAM - ALL stored locally, which means low latency access
- Data copied into S3 with EBS Snapshots
EXAM - Use: Full disk backups of servers
EXAM - Use: Assists with disaster recovery; created EBS volumes in AWS
EXAM - VSM does NOT improve or extend data center capacity
- Something something iSCSI

### Storage Gateway - Volume Cached Mode
Instead of having a local storage volume on prem, you have a local cache. Actual data stored in S3, and does EBS Snapshot backups
EXAM - Use Case: To extend your data center (since data is stored in S3 and only cached locally)

## Storage Gateway - Tape (VTL) - Virtual Tape Library
Storage gateway in VTL mode allows the product to replace a tape based backup solution with one which uses S3 and Glacier rather than physical tape media
- Max size of virtual tape is 5 TiB, same as size of S3 bucket
- Pretends to be an iSCSI tape library, changer, and drive. S3 serves as Virtual Tape Library, Glacier is the Virtual Tape Shelf
- Uses: Backup platform migrations or data center extensions

## Storage Gateway - File
File gateway bridges local file storage over NFS (linux)/SMB(windows) with S3 Storage
- It supports multi site, maintains storage structure, integrates with other AWS products and supports S3 object lifecycle Management
- Files stored this way (to mount point) are visible as objects in an S3 bucket
- Read/Write caching ensure LAN-like performance
- 1 file share + 1 S3 = 1 bucket share. Can have 10 Bucket Shares per file gateway
- Primary data stored in S3
- File Gateway doesn't support Object Locking--use Read Only Mode on other shares or tightly control file access

## Snowball / Edge / Snowmobile [NEW VERSION COMING SOON]
Snowball, Snowball Edge and Snowmobile are three parts of the same product family designed to allow the physical transfer of data between business locations and AWS.

### Snowball
- Order physical device from AWS. Encrypted using KMS. 50TB or 80TB.
EXAM - Range of data to use it for 10TB - 10PB
EXAM - Can order multiple devices / to multiple premises
EXAM - STORAGE ONLY, no compute
### Snowball Edge
EXAM - Has both storage and compute
- Larger capacity VS snowball
- Faster networking with Edge
- 3 types of Edge: 1. Storage Optimized 2. Compute Optimized 3. Compute Optimized with GPU
EXAM - Ideal for remote sites or where data processing on ingestion is needed
### Snowmobile
Portable data center within a shipping container on a truck (mobile, duh)
- Ideal for single location with 10PB+ is required
- 10PB - 100PB
- NOT for multi-site (unless HUGE) or sub 10PB

## Directory Service
What are directories? Identity and asset info storage. Stores objects with structure (inverse tree).
- Multiple trees grouped into a FOREST
- Commonly used with Windows environments
- Users, devices, groups, servers, file shares -- things that can be objects in a directory

 AWS Directory Service is an AWS managed implementation of a directory that runs in a VPC
- High availability, deploy into multiple AZs
- Amazon Workspaces NEEDS the AWS Directory Svc to function
- can be isolated or integrated with an on-premise system

### Directory Service - Three Modes
1. Simple AD - An implementation of Samba 4 (compatibility with basics AD functions)
- cheapest / simplest mode
- 500 - 5000 users
- Simple AD is based off of Samba 4 (an open-source version of Microsoft AD)
- Simple AD designed to be used in isolation
2. AWS Managed Microsoft AD - An actual Microsoft AD DS Implementation
- AWS presense while having an existing on-premises directory
- Primary location is at AWS. TRUST relationships created between AWS and on-premmise
3. AD Connector which proxies requests back to an on-premises directory
- An entity made to integrate with AWS services. It has NO local functionality - great for hosting Workspaces
- If private connectivity fails, AD fails and AWS-side related svd would be interrupted

### When to pick one of the 3 ADs?
Simple AD -  The default. Simple reqs. A directory in AWS
Microsoft AD - Apps in AWS which req MS AD, or you need to TRUST AWS AD Directory Service
AD Connector - To use AWS p&s that require a directory without actually housing one on AWS side (as you have it on prem), use AD connector

## DataSync
AWS DataSync is a product which can orchestrate the movement of large scale data (amounts or files) from on-premises NAS/SAN into AWS or vice-versa
- Data transfer service To and FROM AWS
- Designed to work at HUGE scale
- Keeps metadata (permissions/timestamps)
- Built-in data validation
- Scalable: 10Gbps/agent (~100TB/day)
- FEATURES EVERYWHERE... Bandwidth limiters, incremental/scheduled xfer options, compression and encryption, auto recovery from transit errors, svc integration with S3 EFS FSx, bidirectional transfer
EXAM - The DataSync agent needs to be intalled LOCALLY on-prem
EXAM - communicates via NFS/SMB w/ on-prem storage

### DataSync Components
- tasks: a "job"
- agent: the software living on-prem that reads/writes to on-prem data stores using NFS/SMB
- location: every task has a TO and FROM location

## FSx for Windows Servers
FSx for Windows Servers provides a native windows file system as a service which can be used within AWS, or from on-premises environments via VPN or Direct Connect
- FSx is an advanced shared file system accessible over SMB, and integrates with Active Directory (either managed, or self-hosted).
- It provides advanced features such as VSS, Data de-duplication, backups, encryption at rest and forced encryption in transit.
- Integrats with Directory Service or Self-Managed Active Directory
- Single or Multi-AZ within a VPC
- on-demand/scheduled backups
- accessible using VPC, peering, VPN, Direct Connect
- FSx is dedicated to Windows Environments, the similar EFS is for Linux/Unix
- File level versioning; support volume shadow copies; file-level restores
EXAM - Key words to look for: VSS (user driven restores), native file system over SMB (SMB=windows), uses Windows permissions model, supports DFS (distributed file system), integrates with Directory Service and YOUR OWN directory

## FSx For Lustre
FSx for Lustre is a managed file system which uses the FSx product designed for high performance computing
- It delivers extreme performance for scenarios such as BIG DATA, MACHINE LEARNING and FINANCIAL MODELING
-- FSx for Lustre is a managed Lustre high compute system. Linux clients (POSIX)
- Two deploment types: Persistent or Scratch
-- Scratch: highliy optimized for short-term. No replication and fast. not HA
-- Persistent: Longer term, high avail in one AZ, self-healing
EXAM - If Lustre, POSIX mentioned... FSx for Lustre, ML/big data/fin-modeling

## AWS Transfer Family
Managed file transfer service using these protocols: SFTP, FTP, FTP Transfer, AS2
EXAM - supports transferring data over the following protocols: SFTP, FTPS, and FTP transfer
- Managed File Transfer Workflows (MFTW) - serverless file workflow engine (for tagging and stuff)
- Using FTP? Since not secure, you can only use FTP protocol within VPC (not public)

# SECURITY, DEPLOYMENT & OPERATIONS
## AWS Secrets Manager
AWS Secrets manager is a product which can manage secrets within AWS. There is some overlap between it and the SSM Parameter Store - but Secrets manager is specialised for secrets.
EXAM - Secrets manager is capable of automatic CREDENTIAL ROTATION using LAMBDA.
- For supported services it can even adjust the credentials of the service itself.
- Secrets encrypted using KMS
- Integrates with RDS

EXAM Secrets Manager VS Parameter Store? SM is designed for SECRETS (passwords, API Keys), SM auto rotation of secrets with Lambda,
- Parameter store stores strongs, secure strings--for config info

## Application Layer (L7) Firewall
L7 firewalls are capable of inspecting, filtering and even adjusting data up to Layer 7 of the OSI model. They have visibility of the data inside a L7 connection. For HTTP this means content, headers, DNS names .. for SMTP this would mean visibility of email metadata and for plaintext emails the contents.
- WAF is an L7 Firewall

## Web Application Firewall (WAF), WEBACLs, Rule Groups and Rules
AWS WAF is an L7 web application firewall that helps protect your web applications or APIs against common web exploits and bots that may affect availability, compromise security, or consume excessive resources.
- cross-scripting attacks, SQL Injection, ALLOW/DENY lists, XSS, HTTP Flood, IP Reputation, bots
- Actual unit of configuration within WAF is the WEB Access Control List (ACL)
- Web ACLs use Rules/Rule Groups
-- Rule components: Type, Statement, Action
--- Rule types: Regular or Rate-based
--- Statement: WHAT to match/count... even down to matching body (first 8192 bytes ONLY)
--- Action: Allow, Block, Count, custom respon, label

## AWS Shield
For DDoS protection. AWS Shield provides always-on detection and automatic inline mitigations that minimize application downtime and latency, so there is no need to engage AWS Support to benefit from DDoS protection.

You can use AWS WAF web access control lists (web ACLs) to help minimize the effects of a Distributed Denial of Service (DDoS) attack. For additional protection against DDoS attacks, AWS also provides AWS Shield Standard and AWS Shield Advanced. AWS Shield Standard is automatically included at no extra cost beyond what you already pay for AWS WAF and your other AWS services.
- Shield Standard is free. Automatically provided with CloudFront and R53
- AWS Shield Advanced (not free) provides expanded DDoS attack protection for your Amazon EC2 instances, Elastic Load Balancing load balancers, CloudFront distributions, Route 53 hosted zones, and AWS Global Accelerator standard accelerators.
-- Advanced requires manual configuration, explicit enabling. Has const protection incurred from attacks
--- Advanced Shield USES WAF for L7 DDoS protection

## CloudHSM
CloudHSM - an AWS provided Hardware Security Module products. Similar to KMS... creates/manages/secures crypto keys
- CloudHSM is required to achieve COMPLIANCE with certain SECURITY STANDARDS such as FIPS 140-2 Level 3

### When to use KMS over CloudHSM
- KMS is a SHARED service and AWS has certain level of access to it. KMS is mostly Level 2 FIPS
EXAM - CloudHSM is a SINGLE TENANT HSM (hardware security module). CloudHSM is aws provisioned but fully customer managed. CloudHSM is LEVEL 3 FIPS compliant.
* EXAM - Need CloudHSM if need access by industry standard APIs (not AWS api's); JCE, PKCS#11, CryptoNG
EXAM - Need to enable Transparent Data Encryption (TDE) for Oracle DBs? CloudHSM
EXAM - Protect private keys for an issuing CA (cert authority)

## AWS Config
AWS Config is a service which records the configuration of resources over time (configuration items) into configuration histories.
- All the information is stored regionally in an S3 config bucket.
- AWS Config is capable of checking for compliance .. and generating notifications and events based on compliance.
- Terms: config item, config history, config rules, config stream, config notifications (via eventbridge or sns)
### AWS Config - Two main jobs
1. Record configuration changes over time on resources
2. Auditing of changes, compliance with standards
NOTE: AWS Config DOES NOT prevent changes from happening; no protectoin, just watches

## Amazon Macie
Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS.
- protect personally identifiable information of stuff in S3 buckets
- Can have Managed (built in, ML\patterns) or Custom Data Identifiers (regex based)
- checks s3 buckets

## Amazon Inspector
Inspect for COMPLIANCE. Amazon Inspector is an automated security assessment service that helps improve the security and compliance of applications deployed on AWS. Amazon Inspector automatically assesses applications for exposure, vulnerabilities, and deviations from best practices
- checks EC2 instances and the instance OS, and containers
- creates security reports ordered by priority
- Does Network Reachability (with or without Agent--agent means more info)
- Packages: Host Assessments, Common Vulnerabilities/Exposures (CVE), Center for Internet Security (CIS) Benchmarks, Security Best Practices for Amazon Inspector

## Amazon Guardduty
Guard Duty is an automatic threat detection service which reviews data from supported services and attempts to identify any events outside of the 'norm' for a given AWS account or Accounts.
- CONTINUOUS SECURITY MONITORING SERVICE
- Ingests Logs and Events

### Inspector VS GuardDuty: If we try to describe it in a chronological fashion, you can have Inspector set up at the start when you deploy your applications, and then GuardDuty immediately after that in order to receive alerts on potential threats. Amazon Inspector provides you with security assessments of your applications’ settings and configurations while Amazon GuardDuty helps with analysing the entirety of your AWS accounts for potential threats.


# Infrastructure as Code (CloudFormation)
## CloudFormation Physical & Logical Resources
- CloudFormation defines logical resources within TEMPLATES (using YAML or JSON).
-- The logical resource defines the WHAT, and leaves the HOW up to the CFN product.
- A CFN stack creates a physical resource for every logical resource - updating or deleting them as a template changes.
- TEMPLATES create STACKS. A stack creates/modifies/deletes physical resources based on the logical resources in the template

## CloudFormation - Template and Pseudo Parameters
Template and Pseudo Parameters are two methods to provide INPUT to a template, which can influence what resources are provisioned, and the configuration of those resources.
- Pseudo parameters. These params are injected and made available by AWS. Template params are user made with YAML/JSON

## CloudFormation Intrinsic Functions
AWS CloudFormation provides several built-in functions that help you manage your stacks. Use intrinsic functions in your templates to assign values to properties that are not available until runtime.
- !Ref & Fn::GetAtt. Ref is primary value, attributes are secondaries. !GetAtt
- Fn::Join & Fn::Split
- Fn::GetAZs & Fn::Select
- Conditions Fn::IF,And,Equals,Not,Or
- Fn::Base64 & Fn::Sub. Base64 outputs base64 encoded text
- Fn::Cidr. Creating a cider range based on parent VPC CIDR range

## CloudFormation Mappings
The optional Mappings section matches a key to a corresponding set of named values. For example, if you want to set values based on a region, you can create a mapping that uses the region name as a key and contains the values you want to specify for each specific region
- Use the Fn::FindInMap intrinsic function to retrieve values in a map.
- Mapping -> key:value
- Mappings Improves Template Portability

## CloudFormation Outputs
The optional Outputs section declares output values that you can import into other stacks (to create cross-stack references), return in response (to describe stack calls), or view on the AWS CloudFormation console.
- For example, you can output the S3 bucket name for a stack to make the bucket easier to find.
- Output section is optional
- Visible as outputs from CLI, console UI, parent stack, can be exported to other stacks (cross-stack references)
- Components: Description and Value

## DEMO - Template v2 - Portable - Skipping for now

## CloudFormation Conditions
The optional Conditions section contains statements that define the circumstances under which entities are created or configured. You might use conditions when you want to reuse a template that can create resources in different contexts, such as a test environment versus a production environment.
- Conditions evaluated to True or False and are processed BEFORE resources are created
- In your template, you can add an EnvironmentType input parameter, which accepts either prod or test as inputs.
- Conditions are evaluated based on predefined pseudo parameters or input parameter values that you specify when you create or update a stack.
- Within each condition, you can reference another condition, a parameter value, or a mapping.
- Components: Parameter, Conditions, Resources

## CloudFormation DependsOn
With the DependsOn attribute you can specify that the creation of a specific resource follows another. When you add a DependsOn attribute to a resource, that resource is created only after the creation of the resource specified in theDependsOn attribute.

## CloudFormation Wait Conditions & cfn-signal
CreationPolicy, WaitConditions and cfn-signal can all be used together to prevent the status if a resource from reaching create complete until AWS CloudFormation receives a specified number of success signals or the timeout period is exceeded.
- The cfn-signal helper script signals AWS CloudFormation to indicate whether Amazon EC2 instances have been successfully created or updated.
- After you define all your conditions, you can associate them with resources and resource properties in the Resources and Outputs sections of a template
- tl;dr resource create paused until expected signal(s) received


## CloudFormation Nested Stacks
Nested stacks allow for a hierarchy of related templates to be combined to form a single product
- A ROOT STACK can contain and create nested stacks .. each of which can be passed parameters and provide back outputs.
- Nested stacks should be used when the resources being provisioned ****share a lifecycle**** and are related.
- Single stacks have resource limits of 500, use nested to overcome this
- Stacks are isolated; can't easily reference each other. Nested stacked can reference outputs from parents, however
- Nested stacks for modular-type template re-use. Cross-reference stacks are for resource re-use

## CloudFormation CloudFormation Cross-Stack References
- Nested stacks for modular-type template re-use. CloudFormation Cross-Stack References are for resource re-use
-- Outputs in one stack reference logical resources or attributes in that stack
-- Outputs can be exported, and then using the !ImportValue intrinsic function, referenced from another stack.
--- Exports must be UNIQUELY NAMED in region

## CloudFormation Stack Sets
StackSets are a feature of CloudFormation allowing infrastructure to be deployed and managed across MULTIPLE REGIONS and MULTIPLE ACCOUNTS from a single location.
- Additionally it adds a dynamic architecture - allowing automatic operations based on accounts being added or removed from the scope of a StackSet.
- StackSets are containers in an admin account which contain stack instances which REFERENCE stacks
-- Stack instances and stacks are in TARGET ACCOUNTS
- Each stack is 1 region 1 account

### CFN StackSet Terms
- Concurrent Accounts. How many concurrent stacks are created together. Eg. If you have 10 stacks to make, you set concurrnet acct to 2... you'll create stacks in 5 pairs.
- Failure Tolerance. Amount of individual deployments that can fail before StackSet is considered failed
- Retain Stacks. Delete stacks FROM YOUR stack set, but save them so they continue to RUN INDEPENDENTLY of your stack set by choosing the Retain Stacks option. You can then manage retained stacks outside of your stack set in AWS CloudFormation. "cut it loose"

## CloudFormation Deletion Policy
With the DeletionPolicy attribute you can preserve or (in some cases) backup a resource when its stack is deleted. You specify a DeletionPolicy attribute for each resource that you want to control. If a resource has no DeletionPolicy attribute, AWS CloudFormation deletes the resource by default.
- cut the resource loose from the Stack b/c resource deletion can sometimes cause data loss
- Deletion Policies: Delete, Retain, or Snapshot (if supported)
-- Snapshots continue past stack lifetime

## CloudFormation Stack Roles
Stack roles allow an IAM role to be passed into the stack via PassRole
- A stack uses this role, rather than the identity interacting with the stack to create, update and delete AWS resources.
- It allows ROLE SEPARATION and is a powerful security feature

## CloudFormation Init (CFN-INIT)
CloudFormationInit and cfn-init are tools which allow a desired state configuration management system to be implemented within CloudFormation
- Another way to provide config info into EC2 instance
- Use the AWS::CloudFormation::Init type to include metadata on an Amazon EC2 instance for the cfn-init helper script. If your template calls the cfn-init script, the script looks for resource metadata rooted in the AWS::CloudFormation::Init metadata key. cfn-init supports all metadata types for Linux systems & It supports some metadata types for Windows
- AWS::CloudFormation::Init which is procedural (you tell it how to bootstrap itself)
-... or CFN-INIT which is where you tell the system WHAT you want and it builds it out (this one is idempotent). It is run ONCE as a part of bootstrapping; if CloudFormation::Init is updated, cfn-init isn't re-run (which is where cfn-hup comes in, see below)

## CloudFormation cfn-hup
The cfn-hup helper is a daemon that detects changes in resource metadata and runs user-specified actions when a change is detected (Ie. re-running cfn-init). This allows you to make configuration updates on your running Amazon EC2 instances through the UpdateStack API action.

## CloudFormation ChangeSets - CFN stack update dry runs (like Find/Replace dry runs in WordPress)
When you need to update a stack, understanding how your changes will affect running resources before you implement them can help you update stacks with confidence. Change sets allow you to preview how proposed changes to a stack might impact your running resources, for example, whether your changes will delete or replace any critical resources, AWS CloudFormation makes the changes to your stack only when you decide to execute the change set, allowing you to decide whether to proceed with your proposed changes or explore other changes by creating another change set.

## CloudFormation Custom Resources
Custom resources enable you to write custom provisioning logic in templates that AWS CloudFormation runs anytime you create, update (if you changed the custom resource), or delete stacks
- AKA Custom Resources let CFN integrate with anything it doesn't yet, or doesn't natively support

# NOSQL Databases & DynamoDB
## DynamoDB - Architecture
DynamoDB is a NoSQL fully managed Database-as-a-Service (DBaaS) product available within AWS. This is the traditional serverless DB route in AWS
- uses Key/Value &/or Document data. "Wide column DB"
- no self-managed servers or infrastructure
- Highly resilient and can be optionally Globally resilient
- Single digit millisecond speeds -- SSD based
- manual or auto scaling or on-demand (set and forget)
- RCU - read capacity units, WCU - write capacity units

### DynamoDB - Tables
- Table is a grouping of ITEMS with the same PRIMARY key. Item is like a row in a traditional DB
-- item max size 400KB. item can have all, none, or some of the attibutes/columns in an item(row)
- Table has two primary keys to pick from:
1. Partition key. made up of just a partition key
2. Composite primary key. made up of a partition key and a sort key

### DynamoDB - Backups
Two types of backups:
1. On-demand backup. Full copy of table retained until removed
2. Point-in-time Recovery. Not enabled by default. 35 day window to restore to 1 second granularity, PER table restores
EXAM - NoSQL mentioned? DynamoDB
EXAM - Key/Value DB mentioned? DynamoDB
EXAM - Relational Data/DB? NOT DynamoDB

## DynamoDB - Operations, Consistency and Performance
Key elements of READS and WRITES to DynamoDB and step through how the QUERY AND SCAN operations work.
- When you create a DynamoDB table, you choose from two capacity modes: on-demand and provisioned
-- On-Demand: If you need low admin overhead or traffic is unknown/unpredictable. Price per million R or W units. More expensive
-- Provisioned: RCU/WCU capacity set per table basis
- 1 RCU = 1 x 4KB read ops per second, 1 WCU is 1 x 1KB write ops per second
-- Every table has RCU/WCU Burst Pool (300 seconds)

### DynamoDB - Query and Scan
- Query. One way to retrieve data. Accepts a single PK value and optionally an SK or range.
- Scan. Least efficient but most flexible way to get data in DynamoDB. Scans through table item-by-item - consumed capacity is all items read/scanned

### DynamoDB - Consistency model in DynamoDB
In DynamoDB, each piece of data is replicated into different AZs known as Storage Nodes, and one is the Leader storage node. Leader storage nodes receive the reads/writes and write updates are replicated by leader to other nodes
- Eventually Consistent Reads. You are not guaranteed to see the most updated data if you're send to a node that hasn't received the replicated new data yet. Cheaper option
- Strongly Consistent Reads. Always reads from Leader node to guarantee data up to date
NOT: Not every app can tolerate eventual consistency (some things MUST be accurate)

### Calculate WCU
Problem: Need to store 10 items per second, 2.5K average size per item
1. Round up average size to nearest whole number. 2.5K -> 3
2. Multiply item size x # items/sec. 3 x 10 = 30
= 30 WCU required

### Calculate RCU
Problem: Need to retrieve 10 items per second, 2.5K average size per item
1. Divide Item size / 4 then round up. 2.5K/4K rounded up is 1.
2. Multiply item size x # items/sec. 1 x 10 = 10
= 10 RCU required for Strongly Consistent. Eventual is 50% cost, so 5RCU required for Eventually Consistent Reads

## DynamoDB - Local and Global Secondary Indexes
Local Secondary Indexes (LSI) and Global Secondary Indexes (GSI) allow for an alternative presentation of data stored in a base table.
- Indexes improve the efficiency of operations in DynamoDB
- LSI allow for alternative Sort Key's (SAME PK) whereas with GSIs you can use alternative PK and SK.
-- LSIs only created at Table creation
- GSIs can be created any time. 20 limit of GSIs per base table. Uses alternate PK and SK. GSIs are ALWAYS eventually consistent, so to use GSI your tables needs to be able to cope with eventualy consistency
NOTE: Use GSI as default and LSI only when STRONG consistency index is required

## DynamoDB - Streams & Lambda Triggers
DynamoDB Streams are a 24 hour rolling window of time ordered list of changes to ITEMS in a DynamoDB table
- Streams have to be enabled on a per table basis , and have 4 view types:
1. KEYS_ONLY. Only keys of item that changed
2. NEW_IMAGE. New item after change
3. OLD_IMAGE. Old item before change
4. NEW_AND_OLD_IMAGES. Both old and new states
- Lambda can be integrated to provide trigger functionality - invoking when new entries are added on the stream.

### DDB - Triggers
- Streams are the foundation of Triggers
- Triggers allow fo actions to take place in the event of data change, use can use Lambda to perform a compute action in response

## DynamoDB - Global Tables
DynamoDB Global Tables provides multi-master global replication of DynamoDB tables which can be used for performance, High Avail or Disaster Recovery/Biz Continuity reasons.
- GLOBAL table and then multiple regions with tables becoming REPLICA tables
- LAST WRITER WINS is used for conflict resolution; the most recent Write is what is replicated to other tables
- Global app must tolerate eventual consistency. However, region reads in the same region as writes are strongly consistent

## DynamoDB - Accelerator (DAX)
DynamoDB Accelerator (DAX) is an in-memory cache designed specifically for DynamoDB. It should be your default choice for any DynamoDB caching related questions.
- Global table has sub-second replication between table replicas
- DAX is less overhead than normal caching
- Can scale up or out (bigger or more DAX instances)
- Supports "write-through"
- Good for read-heavy work loads

## DynamoDB - TTL
Amazon DynamoDB Time to Live (TTL) allows you to define a per-item timestamp to determine when an item is no longer needed.
- Shortly after the date and time of the specified timestamp, DynamoDB deletes the item from your table without consuming any write throughput.
- TTL is provided at no extra cost as a means to reduce stored data volumes by retaining only the items that remain current for your workload’s needs.

## Amazon Athena
Amazon Athena is serverless querying service which allows for ad-hoc questions where billing is based on the amount of data consumed.
- Athena is an underrated service capable of working with unstructured, semi-structured or structured data.
- tl;dr take data stored in S3 and perform ad-hoc queries on that data. Pay only for the data consumed while running query (and s3 storage)
- Uses process called "schema-on-read", a table-like translation --- original data on S3 never changed
EXAM - Best option for querying AWS logs; VPC flow logs, CloudTrail, ELB Logs, cost reports, Glue, Web Server, etc
- Athena Federated Query -- a way for Athena to query outside S3
- Athena = ad-hoc. Redshift is not ad-hoc

## Elasticache
Elasticache is a managed in-memory cache which provides a managed implementation of the redis or memcacheD engines. Useful for READ-HEAVY workloads that demand low latency, scaling reads in a cost effective way and allowing for externally hosted user session state
- in-memory database for high performance
- Two engines:
1. Redis. supports advanced data structures. Multi-AZ. Backup and restore.
2. Memcached. Supports simple data structures. No backups. Multi-threaded
- reduces database workloads
EXAM - Can store Session Data (for stateless servers)
EXAM - Elasticash requires application code changes to work

## Redshift Architecture
Redshift is a column based, petabyte-scale, data warehousing product within AWS
- It's designed for OLAP products within AWS/on-premises to add data to for long term processing, aggregation and trending. (Not OLTP which is row/transaction; inserts/modifies/deletes)
- Redshift Spectrum. Direct Query S3 without having to load it into redshift in advance
- Federated Query. Query data in remote data sources
- Redshift has Enhanced VPC Routing; VPC networking for advanced networking control
- AZ specific

## Redshift DR and Resilience
- RedShift only exists in 1 AZ. There are a number of recovery features:
- snapshots. Either automatic (1-35 day retention) or manual (no retention period). Since data is backed up to S3, you have S3 protections against failure
-- snapshots can be sent to other regions for disaster recovery with a separate configurable retention period
- SERVER based

# MACHINE LEARNING
## Amazon Comprehend
Amazon Comprehend is a natural-language processing (NLP) service that uses machine learning to uncover valuable insights and connections in text.
- input document and it parses and develops insights (like name, addresses, PII, phrases, common elements, sentiment)
- pre-trained or custom models
- realtime for small workloads, async for larger workloads
- confidence 0.0 - 1.0 (1 being wholly confident)

## Amazon Kendra
Amazon Kendra is an intelligent search service powered by machine learning (ML).
- designed to mimic interacting with a human expert
- suppports wide range of question types: factoid, descriptive, keyword
- Kendra is a backend service that you'd API-integrate with your app for search functionality
### Kendra Concepts
- Index. Searchable data
- Data source. Where your data lives; S3, Google Workspace, RDS, Salesforce, FSx, etc etc
- Documents. Structured and unstructured indexable items

## Amazon Lex
Amazon Lex is a fully managed artificial intelligence (AI) service with advanced natural language models to design, build, test, and deploy CONVERSATIONAL interfaces in applications.
- interactive chat bots
- backend service that you'll use to add capabilities to your application
- POWERS THE ALEXA SERVICE
- Automatic Speesh Recognition (ASR); sppech-to-text, Natural Language Understanding (NLU); intent,
- Chatbots, Voice Assistance, Q&A Bots, Phone bots

## Amazon Polly
Amazon Polly is a service that turns text into lifelike speech text to speech TTS), allowing you to create applications that talk, and build entirely new categories of speech-enabled products.
- polly - parroting text to speech. NO TRANSLATION
- two types of TTS.. .standard and neural. Standard concatenates phenomes, neral much more complex but more natural

## Amazon Rekognition
Amazon Rekognition offers pre-trained and customizable computer vision (CV) capabilities to extract information and insights from your IMAGES and VIDEOS.
- deep learning based
- ID objects, people, text, activities, face detection/analysis/comparison, pathing, much more
- Can analyze live video by integrating with kinesis video streams

## Amazon Textract
Amazon Textract is a machine learning (ML) service that automatically extracts text, handwriting, and data from scanned documents. It goes beyond simple optical character recognition (OCR) to identify, understand, and extract data from forms and tables
- input documents = JPG, PNG, PDF, TIFF
- output: extracted text, structure, and analysis
- types of analysis: document, receipts, identity documents
- backend style service that can be intergrated with your apps or with other AWS services

## Amazon Transcribe
Amazon Transcribe is an automatic speech recognition (ASR) service that uses machine learning models to convert audio to text. You can use Amazon Transcribe as a standalone transcription service or to add speech-to-text capabilities to any application.
- input Audio, output Text
- Use Cases: full text indexing to allow searching of audio, create meeting notes, subtitles/captions/transcripts
-- call analytics (sentiment, characteristics, etc)

## Amazon Translate
Amazon Translate is a neural machine translation service that delivers fast, high-quality, affordable, and customizable language translation.
- one word at a time translation and outputs semantic representation or meaning, decorder reads meaning and writes target language
- backend style service that integrates with other services/apps/platforms

## Amazon Forecast
Amazon Forecast is a fully managed service that uses statistical and machine learning algorithms to deliver highly accurate time-series forecasts.
- Predicting retail demand, supply chain, staffing, web traffic.
- Import historical & related data, understand what's normal, output forecasts and forecast explainability
- backend style service that integrates with your apps or you can use web console, CLI. And APIs, Python SDK

## Amazon Fraud Detecto
Amazon Fraud Detector is a fully managed fraud detection service that automates the detection of potentially fraudulent activities online. These activities include unauthorized transactions and the creation of fake accounts. Amazon Fraud Detector works by using machine learning to analyze your data.
- online fraud, transactional fraud, account takeover (phishing etc)
- backend style service that integrates into your apps

## Amazon SageMaker
Amazon SageMaker is a fully managed machine learning service. With SageMaker, data scientists and developers can quickly and easily build and train machine learning models, and then directly deploy them into a production-ready hosted environment.
- implementation of a fully managed ML service: fetch, clean, prepare, train, eval, deploy, monitr/collect -- the whole ML lifecycle
- SageMager Studio - Dev environment for the ML lifecycle
- SageMaker Domain - olation or groupings / container for a project
- Containers. Docker containers deployed to ML EC2 instances
- Hosting. Can host ML models
- SageMaker has no cost, the resources it creates do

## AWS Local Zones
AWS Local Zones are a type of infrastructure deployment that places compute, storage, database, and other select AWS services close to large population and industry centers.
- physical distance affects latency and performance. AWS Local Zones brings infrastructure physically closer for super low latencies
- Local Zones support DX (direct connect)
- Local Zones have Private networking with their parent region
- Not all products support Local Zones
- HIGHEST PERFORMANCE REQUIRED? Local Zones

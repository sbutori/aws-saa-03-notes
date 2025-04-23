# S3 Security (Resource Policies & ACLs)

Bucket Policies: Type of AWS Resource Policy

EXAM: S3 is PRIVATE by default
- only ID with any initial access is Account Root User of account which owns the S3 Bucket
- Any other permissions must be explicity granted through a few ways:
-- S3 Bucket Policy, a form of Resource Policies. RP's: These are attached to Resources, not Identities
-- Resource Policies (RP) provide a resource perspective on permissions. Control access on a Resource
--- Controlling who can access a resource
- EXAM: Unlike ID Policies, Resource Policies's have presence of an explicit Principal Component.

NOTE: You can only attach IDP's to ID's in your OWN account. Only controlling security inside account
- Resource policies can be attached to its own account or DIFFERENT accounts. Inside this RP, you can reference any identities (whether in same or different account)

- RPs can ALLOW/DENY anonymous principals -- even those not authenticated by AWS; anonymous access

### S3 Sec: TL;DR Bucket Policies can grant access for other AWS accounts, and anonymous access

RP's have one major difference to Identity Policies: The presence of an explicit Principal Component.
- The Principal part of an RP defines which Principals are affected by the policy (who is impacted?)

TERM: Principals. A person or application that uses the AWS account root user, an IAM user, or an IAM role to sign in and make requests to AWS. Principals include federated users and assumed roles.
- if a Resource Policy applies to you, you are the Principal
- Eg. "Principal": "*", --> Star is wildcard, which means ANY principal; applies to anyone accessing the Bucket

### S3 Sec: Other s3 Bucket Policy features / functions
Ex 1. Used to control who can access objects, even blocking specific IP addresses
Ex 2. Can Deny a Principal who isn't using MFA from accessing a Bucket
- Additional Examples: https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html

NOTE: There can only be ONE Policy on a bucket. But, this policy can have multiple statements.

### S3 Sec: Another Form - Access Control Lists (ACLs) - LEGACY
- Legacy (Use Bucket or Identity Policies instead). This is because ACLs are inflexible; no conditions, for example. Can only do 5 things.
- ACL is a subresource

### S3 Sec: Block Public Access Settings
Added in response to lots of public PR disasters; bad bucket configuration leading to data leaks. Until BPA, public/anons could access Bucket. BPA has setting which override Bucket Policies, BUT they apply to just Public Access
- These settings created when making Bucket, but can be adjusted later

EXAM: Identity Policies: Controlling different resources / prefer IAM / Same account only policies
EXAM: Resource Policies: For just controlling S3 / for anonymous or cross-account access
EXAM: Access Control Lists: NEVER unless required, as they are Legacy. AWS recommends against use of ACLs


## S3 Static Hosting
Accessing S3 is generally done via APIs. Static Website Hosting is a feature of the product which lets you define a HTTP endpoint, set index and error documents and use S3 like a website.
 - S3 Static Hosting allows access to s3 via HTTP - Eg. Blogs
 - Usage (2 Documents required): First enable, and in doing so, you have to set INDEX and ERROR documents
 - Index page is entry point to most websites
 - Error Document: When you access a page that isn't there, or other type of service side error that's when Error Doc is shown
 -- Index and Error docs need to be html documents as s3 Static Hosting reads HTML files
 - When Static Hosting is enabled, a static website hosting endpoint is created. Name of endpoint is based on bucket name and region
 -- Want a custom Domain? (via R53)... if so, bucket name matters. Name of Bucket MUST match domain name
 --- Eg. Want a site called top10.animals4life.org? Name your bucket "top10.animals4life.org"

 ### S3 Static Hosting: What else is S3 Static Hosting Good for (other than hosting static websites?
 - Example 1: Offloading. If you have a site with lots of images hosted by compute service, you can offload the media to an S3 bucket w/ static hosting to save money (compute is usually pricier). Offload large data to S3.
 - Example 2: Out-of-band pages. In IT, this means method of accessing something outside of the main way. Example: You want to perform server maintenance and the server needs to go down. You can put an "Under Construction" static page in S3 to put up when production server is down.

 ## S3 Pricing
 Pricing for S3 forms of a number of major components:
 - Cost to Store Data: per gigabyte per month charge
 - Data Transfer Free: for every Gb of data transferred out of S3, per gigabyte charge
 - Data Requests: GET, POST, list, port. Cost per 1,000 operations

### S3: What's FREE?
 - 5Gb monthly storage
 - 20,000 GET requests
 - 2,000 PUT requests


## DEMO Creating a Static Website with S3
 1. Create S3 Bucket > bucket name "[custom-globally-unique]" > uncheck "Block All Public Access" and acknowledge > Create Bucket
 2. Enable Static Website Hosting > access new bucket > Properties tab > scroll to bottom, Edit Static Website Hosting, Enable > Hosting Type: Static > Index Document "index.html" > Error Document "error.html" > In properties, scroll down and copy new static URL
 3. Upload some Objects to the bucket > Objects tab, "Upload" > Upload index.html, error.html, and "img" folder
 4. Paste copied URL into browser
 - You'll get a 403 Forbidden error (Access Denied) - Remember, S3 buckets are private by Default. We need to add permissions for anonymous/public users to visit site (no method to provide creds to S3)
 5. Give Permissions to un-authenticated uses to acess bucket > Access S3 bucket > Permissions tab > Bucket Policy > Edit > Grab JSON from .zip > Paste in, but replace the "Resource" value BEFORE the "/*" (mine now looks like "Resource":["arn:aws:s3:::top10catsever876.com/*"]) > Save
 6. You can now visit your website
 Extra: If you Registered a domain in R53, you can customize your URL. R53 > Hosted Zones > access your domain > Create Record > select Simple Routing > Record Name Eg. "top10".animals4life.io > choose Endpoint "Alias to S3 website endpoint" > Region: us-east-1 > S3 endpoint, select your bucket > click "Define simple record" > Create Records. Now Cantrill can go to top10.animals4life.io. Remember: Domain name and bucket name must match exactly to do this
 7. Clean Up: Empty bucket, delete bucket


 ## Object Versioning & MFA Delete (EXAM)
- Object versioning is a feature which can be enabled on an S3 bucket - allowing the bucket to store multi versions of objects
- These objects can be referenced by their version ID to interact directly - or omit this to reference the latest version of an object
- Objects aren't deleted - object deletion markers are put in place to hide objects.
- Object Versioning starts off in a Disabled state. Once enabled, you CANNOT disable it again. Instead, you can Suspend it. A suspended bucket can be re-enabled.

EXAM: A bucket with Object Versioning enabled can NEVER have OV disabled again

Without versioning, Objects are identified solely by the Object's name, which is unique in the bucket. If you modify an object, the original version of the object is replaced. Versioning lets you store multiple versions of objects within a bucket/
- Operations which would modify an Object instead creates a new version
- Another attribute on an Object in a Bucket (other than Key (name)) is 'id'. A bucket without Versioning enabled has Object id's of 'null'
-- With Versioning enabled, id's are active. Modifying an object will create a new object with a new id. The newest Object version is called the Current or Latest Version
-- If an object is accessed without indicating to S3 which version is required, the Latest Version will be returned. You can access versions by specifying the id

### Object Versioning: Versioning Impacts Deletions
With Versioning enabled, if we indicate to S3 we want to delete the object without giving Version ID, S3 will create a new version of the object with a Delete Marker. In reality, the Object is just hidden. You CAN delete a Delete Marker, which re-activates previous versions.
- To truly delete a particular object, you need to specify the id

### Object Versioning: REMINDERS:
- Versioning CANNOT be disabled after activating, only suspended (and suspending doesn't remove old versions, so you're still billed for all of it)
- Space is consumed by all versions of the object and you are billed for ALL versions
- Only way to return to reduced cost due to Versioning is to delete the bucket and re-add all content to another bucket that doesn't have Versioning enabled

### MFA Delete
Something enabled withing the Versioning config in a bucket. This makes MFA required to change a bucket's versioning state (To change from Enabled to Suspended, etc).
- MFA is required to Delete Versions. When performing API calls to change/delete Version, you need to provide serial number of MFA token AND code, concatenated


## DEMO S3 Versioning
This lesson looks at a 'Animal of the week' website, and how to use versioning to recover when objects are changed and deleted accidentally or intentionally.

1. Create new S3 Bucket: Login to Mgmt/General Admin account > S3 > Buckets > Create Bucket > name [unique] "acbucket23425" > uncheck Block Public Access > Enable Bucket Versioning > Create Bucket
2. Enable Static Hosting: New bucket > Properties tab > Enable Static Website Hosting (bottom of page) > Index doc "index.html" > error doc "error.html" > Save changes
- REMEMBER: Just enabling Static Website Hosting is insufficient, you also need to add a Bucket Policy
3. Add Policy: Access new bucket > Permissions tab > edit Bucket Policy > copy text from bucket_policy.json (from demo .zip) > paste in json, edit Resource to look like " "Resource": "arn:aws:s3:::acbucket23425/*" ", except with your copied Bucket ARN instead
4. Upload files: acbucket > Upload > file "index.html" and folder "img" > click Upload
5. Versions: acbucket > Objects tab > toggle Show Versions > access winkie.jpg > Upload > Add Files > version 2/winkie.jpg > Upload > Access winkie.job object and see multiple versions if "Show versions" toggle is enabled
6. Delete Marker: Untoggle Show Versions within winkie.jpg Object > select winkie.jog > Delete > toggle on Show Versions > see Delete Marker > select Delete Marker and Delete
7. Delete Latest Version to Perma-Delete: acbucket Objects > toggle on Show Versions > select Latest Version of winkie.jpg > Delete
- NOTE: Whenever working with specific Versions of an object, this permanently deletes and makes next most recent version the new Latest / Current Version
8. Clean Up: Empty acbucket > Delete acbucket

REMEMBER: Versioning Enabled can incur significantly higher costs than Versioing turned off.


## S3 Performance Optimization
Single PUT Upload: Up to 5GB Max (don't trust single PUT with this much data), 1 blob of data. Single stream transfer uploaded using s3:PutObject API PUT call. If errors out/fails, full data stream has to restart, making it unreliable. Speed and reliability affected by this sub-optimal transfer.

Multipart Upload: MINIMUM size 100MB. Improves speed and reliability compared to Single Put Upload. Breaks data stream into parts which can break and restart separately, making it more resilient. Always choose multipart beyond 100MB upload.
- Upload can be split into max of 10,000 parts, parts ranging from 5MB -> 5GB. Each part can fail and restart in isolation; risk of uploading large amounts of data is reduce. Transfer rate is increased as you are uploading in parallel, more effectively using internet bandwidth
- The last remaining part can be less than 5MB

S3 Transfer Acceleration:
- How Global Transfer works: Data does not travel in the most efficient route, data can take indirect paths and potentially slows down from hop to hop.
- S3 Transfer acceleration selects the closest, best-performing AWS Edge location and sends directly to S3 bucket. Edge Location transits data over AWS Global Network instead of via public internet (public internet is flexible and reliable over fast). Internet is like public transit with many layovers, whereas AWS Global Network is like a direct flight.
- By Default, Transfer Acceleration is Disabled on an S3 Bucket
- Bucket naming restrictions for Transfer Acceleration: no periods in name, must be DNS compatible


## DEMO S3 Performance
Enable Accelerated Transfer on an S3 bucket and review the AWS provided tool to complete Direct uploads vs Accelerated Transfer

Create s3 bucket (no periods in name, DNS naming compatible) > name "testac3464576" > Create Bucket > new Bucket Properties tab > edit Transfer Acceleration, Enable > Save changes
- NOTE: When Transfer Acceleration is Enabled, an Accelerated endpoint address is provided. You need to use this endpoint to get the benefit of Accel. Transfers.
- To compare upload speeds: http://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html


## Key Management Service (KMS)
AWS Key Management Service (AWS KMS) makes it easy for you to create and manage cryptographic keys and control their use across a wide range of AWS services and in your applications. AWS KMS is a secure and resilient service that uses hardware security modules that have been validated under FIPS 140-2, or are in the process of being validated, to protect your keys.
- Regional & Public Service: every region isolated when using KMS. Note: KMS is capable of multizone (covered later)
- Public service, occupies AWS Public Zone and can connect from anywhere with access to this zone, with permissions
- KMS Mgmt allows you to CREATE, STORE, MANAGE keys
- Has both Symmetric and Asymmetric keys
- Capable of cryptographic operations, ENCRYPT, DECRYPT, etc
- KMS Keys NEVER leave the product; this is it's primary function. To keep keys in and securely held WITHIN the service

EXAM: FIPS 140-2: KMS Provides a service compliant with FIPS 140-2 Level 2, a US security standard
EXAM: KMS is REGIONAL and PUBLIC

### KMS: KMS Keys
Used by KMS within cryptographic operations. You, apps, and other AWS svcs can use these keys. KMS Key is a container for physical key material; it is Logical (representation of an encryption key)--the data contained is ID, date, policy, description, and state
- Every KMS Key backed by Physical Material (Actual key that is used to encrypt/decrypt). Physical Key material can be generated by KMS or imported
-- This 'material' is what is used to direct/encrypt (data up to 4KB)
- After picking region, user first performs CreateKey. Reminder: KMS performs all operations internally (KMS Keys NEVER leave KMS)
- TERM: Role Separation: Permissions to Encrypt, Decrypt, and Generate Key are all SEPARATE permissions

### KMS: Data Encryption Keys (DEK)
Earlier, it was noted that KMS Keys can only perform cryptoOps on data 4KB or less in size. KMS gets around this by generating Data Encyption Keys
- DEK is generated using KMS key using GenerateDataKey operation
- *** KMS doesn't store the DEK in any way. It provides it to you, then discards it. KMS does not do the encyption or decryption of data using DEKs, you or the service performs the DEK encrypt/decrypt.

### KMS: Key Concepts
- KMS Keys are ISOLATED to a REGION and never leave
- Multi-region keys discussed in a different lesson
- Within KMS, keys are either AWS owned or Customer owned. AWS Owned are for use in multiple accounts and operate in the background
-- Two types of Customer Owner 1) AWS Managed (auto-created by AWS services, like S3) 2) Customer Managed (created explicitly by customer to be used by app or AWS svc
-- Customer Managed Customer Owned KMS Keys: More configurable. AWS Managed can't really be customized
- Aliases: shortcuts to keys. Aliases are PER REGION. Neither aliases or keys are Global, by default

### KMS: Key Rotation
When the physical backing material used to perform cryptoOperations is changed at regular intervals (usually once annually)
- KMS Keys support key rotation
- Customer Key Rotation is optional
- When key is rotated, material (data) is changed and NOT the logical key

### KMS: Permissions (Policies / Security)
Account Trust is explicitly added to a Key Policy, or not.
- Key Policy (type of Resource Policy): Every KMS key has a policy. On Customer Managed keys, you can change the policy
- Keys have to explicitly be told they trust the account they're contained within
- Generally, KMS is managed using this combination: Key Policies trusting account + Identity Policies allowing IAM users to interact with key

REMINDER: Resource Policies require a PRINCIPAL in their JSON file, IDP's do not


## DEMO - KMS - Encrypting the battleplans with KMS
Run through the practical steps of creating and configuring a KMS Key, an Alias and we use that Key and the CLI tools to encrypt and decrypt some data

1. Generate KMS Key: Login to Mgmt/General Admin account > KMS > Create a key > Key Type: Symmetric > Next > create alias "catrobot" > Next > Define key admin permissions: check "iamadmin" (so iamadmin user can administer this key) > Next > Define key usage permissions: select "iamadmin" > Next > Review > Finish
2. Enable Key Rotation: Access catrobot key > Key Rotation tab > Check auto-rotate KMS key yearly, Save
3. Use Key / Create Battle Plan: access CloudShell (first icon on right-side main AWS menu, looks like Terminal icon) > Create plaintext battleplan:  || echo "find all the doggos, distract them with the yumz" > battleplans.txt ||
4. Encrypt message - Output will be the Encrypted not_battleplans.enc. Enter below lines into Shell
aws kms encrypt \
    --key-id alias/catrobot \
    --plaintext fileb://battleplans.txt \
    --output text \
    --query CiphertextBlob \
    | base64 --decode > not_battleplans.enc
5. Receiver needs to Decrypt the cihpertext. Enter below lines into Shell:
aws kms decrypt \
    --ciphertext-blob fileb://not_battleplans.enc \
    --output text \
    --query Plaintext | base64 --decode > decryptedplans.txt
6. Clean Up: Select key > Key Actions: Schedule for deletion (waiting period 7-30 days, input 7) > Confirm > Schedule deletion


## SSE-S3 (AES256)
Step through the various encryption options available within S3 and finish by looking at default bucket encryption settings
- Server-Side Encryption (SSE).
-- EXAM: Buckets are NOT encrypted, Objects are; encryption not defined at Bucket level. Encryption is defined at Object level
- Client-Side Encryption (CSE). S3 never sees plaintext

### SSE-S3: SSE VS CSE: How data is stored on disk in an encrypted way; encryption at rest
- Path: Users/App <-> S3 Endpoint <-> S3 Storage.
-- In transit: On both SSE/CSE, Data in transit between User/S3 Endpoint is encrypted in transit.
-- AT REST is what we're focusing on.
--- In CSE, Objects being uploaded are encrypted by the client before they ever leave client-side. AWS will never see data in plain text form. In CSE, you own keys, process, and tooling
--- In SSE, even though data is initially encrypted in-transit using HTTPS, the Objects themselves are NOT initially encrypted. From User > S3 Endpoint, data is in its original form. At S3 endpoint, the data gets encrypted and delivered to S3 Storage as ciphertext.

NOTE: AWS has made SSE mandatory, so you can no longer store objects in an unencrypted form on S3. You control what type of SSE is utilized

### SSE-S3 - Types of Server-Side Encryption within S3:
- SSE-C: SSE with Customer provided/managed Keys. Customer handles keys, S3 handles enc/dec. In-Transit data encrypted by HTTPS, plaintext otherwise
- SSE-S3: [The Default] SSE with Amazon S3 Managed Keys. AWS handles both enc/dec and key mgmt. Uses AES-256
-- Three problems with SSE-S3: 1) Not suitable strongly regulated environments 2) if you need Role Separation, not suitable 3) need to control Key Rotation? not suitable. AKA No Key Control, No Role Separation
- SSE-KMS: SSE with KMS Keys stored in the AWS Key Management Service. KMS service now managing the keys, meaning you can create a Customer Managed Key; isolated permissions, fully configurable. KMS Generates and delivers DE Keys to S3 (KMS does not store DEKs). Great for controlling Permissions, Role Separation and Key Rotation Control.
* Difference between methods is what parts of the process you trust S3 with and how encrypt process/key mgmt is handled
* High Level: Two components to SSE: 1) encrypt/decrypt process, 2) generation/mgmt of crypto keys (these two components handled differently in different SSE types)

### SSE-S3: Resources
- Visual of SSE Methods and Details: https://imgur.com/a/lCg5gmA
- https://docs.aws.amazon.com/AmazonS3/latest/user-guide/default-bucket-encryption.html
- https://docs.aws.amazon.com/kms/latest/developerguide/services-s3.html
- https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html
- https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html
https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html


## Demo - SSE-S3 - Object Encryption and Role Separation
Create an S3 bucket, and upload 3 images to the bucket using different encryption methods

1. Create S3 Bucket > name "catpics[random number]" > Create Bucket
2. Create Key: Nav to KMS > Create Key > Defaults > Alias "catpics" > Next > Don't set any Key Administrative Positions > Next > skip Key Usage Permissions > Finish
3. Upload SSE-S3 image: Back to S3 > catpics bucket > upload just sse-s3-dweez.jpg (must upload separate as we are configuring separate encryptions) > expand Properties accordian, Server-side encryption "Specify an en encryption key" > Encryption settings "Override bucket settings" > Encryption Key type "SSE-S3" > Upload
4. Upload SSE-KMS image w/ AWS managed default key: Upload see-kms-ginny.jpg > Properties > SSE - Specify an encrypt key > Override bucket settings > key type: SSE-KMS > Choose from your AWS KMS keys: choose the one ending "aws/s3", an AWS managed default key > Upload
5. Replace Step 4 Object, SSE-KMS image w/ KMS Generated Key: Upload see-kms-ginny.jpg > Properties > SSE - Specify an encrypt key > Override bucket settings > key type: SSE-KMS > Choose from your AWS KMS keys: alias "catpics" > Upload
6. Apply Deny policy on IAM, preventing KMS use > IAM Dashboard > Users > iamadmin > Permissions tab > Add Policy: Inline Policy > JSON tab > delete template > Paste in JSON provided in lesson > Review policy > name "denyKMS" > Create policy. Now you can open see-s3 image, but not the kms image. IAM > Remove DenyKMS
7. Set Default Bucket Encryption: catpics bucket > Properties tab > Default Encryption "edit" > key type: SSE-KMS > AWS KMS Key: Choose, "catpics"
8. Upload Default Merlin jpg with no encrypt settings: upload default-merlin.jpg without changing anything > Properties tab: Server-side encryption settings - see default stuff added from Step 7

EXAM: If you need to fully manage the keys used as part of S3 encryption process, you have to use SSE-KMS


## S3 Bucket Keys
Amazon S3 Bucket Keys reduce the cost of Amazon S3 server-side encryption using AWS Key Management Service (SSE-KMS). Bucket-level keys for SSE can reduce AWS KMS request costs by up to 99 percent by decreasing the request traffic from Amazon S3 to AWS KMS.
- Without bucket keys, you make a ton of calls to KMS API. Each object is a separate call for DEK to KMS. Also throttling issues--the generate DEK operation can only be run 5,500, 10,000, or 50,000 times per second--this is shared across regions. AKA Using a single KMS Key results in a scaling limit for PUTS per second per key.
-- Bucket Keys improve this situation. Instead of KMS Key generating each DEK, it generates a single time-limited bucket key. This bucket key lives in the bucket and for a period of time, generates DEKs... offloads work from KMS to S3, reducing cost and increasing scalability.
NOTE: Not retroactive, so previously created S3 bucket objects will still be KMS generated DEKs
- CloudTrail (CT) Events will now show the bucket ARN instead of your object ARN
- Offloading KMS to S3 means fewer CT KMS events in the logs
- Bucket keys work with same-region and cross-region replication; the object encryption is maintained. When S3 replicates an encrypted object, it generally preserves the encryption settings
- If replicating plaintext to a bucket using bucket keys, the object is encrypted at the destination side (ETAG changes between source/destination)


## S3 Object Storage Classes
### S3 Standard
### S3 Standard-IA
### S3 One Zone-IA
### S3 Glacier
### S3 Glacier Deep Archive
### Intelligent-Tiering

### Standard
The Default. Stored objects stored across AT LEAST 3 AZs; can cope with multi-AZ failure; this provides 11 9's of durability (if store 10mil objects, on avg you might lose 1 object every 10,000 years).
- Should be used for frequently accessed data that is IMPORTANT and NON-REPLACEABLE
- Replication uses MD5 Checksums and Cyclic Redundancy Checks (CRCs) to detect and fix data corruption
- Successful receipt and storage in S3? S3 API endpoint returns 'HTTP/1.1 200 OK'
- Most balanced class when looking at cost and features
- First byte latency of milliseconds, objects can be made PUBLICLY AVAILABLE (using S3 permissions or static website hosting site for public internet)
#### Standard - Billing
- GB per Month fee for data stored
- $1 per GB charge for transfer OUT (transfer IN is always free)
- Price per 1,000 requests
- NO retrieval fee, NO minimum duration, NO min size
EXAM: S3 Standard should be used for frequently accessed data that is IMPORTANT and NON-REPLACEABLE

### S3 Standard-IA
IA = Infrequent Access. This class is designed for infrequently accessed data that needs reliability and durability.
Data replicated over 3 AZ's. Similar durability and first byte latency. Same checksum and CRCs. Objects can be made publicly available.
### Standard-IA Billing
- Big difference from Standard billing: Storage costs are cheaper than S3 standard
-- New cost component compared to Standard: RETRIEVAL FEE: for every byte retrieved, there is a cost to retrieve IN ADDITION to transfer fee
- MINIMUM DURATION charge: However long you store this class, you're billed minimum of 30 days
- MINIMUM CAPACITY charge: Per object, you are charged as if they are each at least 128KB in size; don't use IA to store lots of tiny objects
- Per request charge, data OUT charge, same as Standard
- Transfer IN is free
EXAM: S3 Standard-IA should be used for LONG-LIVED DATA which is IMPORTANT, but access is INFREQUENT

### S3 One Zone-IA
Similar to Standard-IA in many ways. Cheaper than both Std-IA and Std. The compromise for cost reduction: Data stored using this class is only stored in ONE Availability Zone
- Still retrieval fee, still 30 day minimum billing (like std-IA), still min cap of 128KB (like std-IA)
- One Zone-IA does not provide the multi-AZ resilience model, instead, one AZ used within the region; cheaper storage but greater risk of data loss
- SAME DURABILITY: 11 9's of durability (unless the AZ fails). Data still replicated in the one AZ
EXAM: Used for LONG-LIVED data that is INFREQUENTLY accessed, but for which is NON-CRITICAL and EASILY REPLACED

### S3 Glacier
S3 Glacier, instant access to your data; instant retrieval. Like S3 Standard-IA but cheaper storage, more expensive retrieval, longer minimums. Designed for when you need data instantly, but not very often (like once a month).
- Minimum storage duration charge: 90 days

### S3 Glacier Flexible
Same 3 zone AZ architecture, 11 9's of durability
- Trade-off compared to previous classes:
-- Think of the objects store here as "cold objects"; not ready for use; not immediately available, can't be made public; a RETRIEVAL PROCESS is required
--- Retrieval Jobs, 3 Types (faster = more $:
--- 1. Expedited: data available in 1-5 minutes
--- 2. Standard: available in 3-5 hours
--- 3. Bulk: 5-12 hours
- MINIMUM BILLABLE DURATION: 90 days
- MINIMUM BILLABLE SIZE: 40KB
- No Public Access
EXAM: First Byte latency = minutes or hours. No public access. Glacier Flexible is for situations where you need to store archival data where frequent/real-time access isn't required (like once a year access)

### S3 Glacier Deep Archive
The cheapest form of S3 storage. The most restrictions.
- If Glacier Flexible data is in "chilled" state, Glacier Deep Archive data is "frozen"
- MINIMUM BILLABLE DURATION: 180 days
- MINIMUM BILLABLE SIZE: 40KB
- Objects are not publicly accessible; retrieval job required
-- Retrieval Jobs:
-- 1. Standard - 12 hours
-- 2. Bulk - up to 48 hours
EXAM: First byte latency = hours or days. Used for archival data that is very rarely accessed, if ever. Good for data that requires retention length, or secondary long-term backups

### Intelligent-Tiering
Different from all previous classes; it contains five different storage tiers.
- When you move objects into this class, tier is determined by monitoring the data's usage
- Tiers:
-- Frequent Access (less than 30 days between access)
-- Infrequent Access (30+ days between access)
-- Archive Instant Access (90+ days between access)
-- Archive Access (90 -> 270)
-- Deep Archive (180 -> 730)
Instead of a retrieval cost, Intelligent Tiering has a monitoring and automation cost per 1,000 objects (management fee). Only use this when your app can tolerate asynchronous access patterns.
EXAM: Intelligent-Tiering is designed for long-lived data where usage is CHANGING or UNKNOWN. Ideal for uncertain access and low admin overhead
- https://aws.amazon.com/s3/pricing/
- https://aws.amazon.com/s3/storage-classes/


## S3 Lifecycle Configuration
Allows objects storage classes to be changed and objects deleted automatically. Step through S3 lifecycle management/configuration/rules both in theory and via an S3 console example
- You can create lifecycle rules on S3 buckets which can auto-transition or expire objects in a bucket. Great for objects that have a life cycle (activity on object is predictable)
- Lifecycle Configuration is a set of RULES that consist of ACTIONS. Rules can be applied to a whole Bucket or groups of objects in a bucket if defined by prefixes/tags
- Two types of Actions applied (both work on versions if Object Versions are enabled on bucket):
-- 1. Transition Actions: Change storage class of affected objects
-- 2. Expiration Actions: Delete objects/object versions
EXAM: S3 ONE ZONE-IA CANNOT transition into S3 Glacier-Instant Retrieval. Transition can't go up the waterfall of classes, only down. See waterfall visual
EXAM: Smaller objects can cost MORE (minimum size)
EXAM: If object initially placed in Standard class, there is 30 day minimum period where an object needs to remain on S3 Standard before transition down the waterfall
EXAM: A SINGLE rule CANNOT transition to Standard-IA or One Zone-IA and THEN to glacier classes within 30 days (duration minimum); must remain in IA class for min 30 days before moving to Glacier


## S3 Replication
S3 has two replication features which allow objects to be replicated between SOURCE and DESTINATION buckets in the same or different AWS accounts

### S3 Replication - Two Types:
- Cross-Region Replication (CRR) is the process used when Source and Destination are in different AWS regions
- Same-Region Replication (SRR) is used when the buckets are in the same region.

### S3 Replication - The Difference:
With Cross-Region Replication, The Destination Bucket, b/c it's in a different AWS account, doesn't trust the Source account or its IAM Role that gets created for Replication purposes by Source Bucket. If configuring replication between different accounts, you have to create a Bucket Policy in the Destination Bucket which allows the Source IAM role to write/replicate into the Destination bucket.

### S3 Replication - Options:
1st option: What you replicate. Default is entire Source bucket to Destination bucket (everything in it), OR subset.
- For subset, create a rule that adds a filter to filter objects by prefix or tags
2nd: Storage Class to be used. The Default is to use the same class, but you can choose a cheaper class if this data is secondary.
EXAM: Default Storage Class to use for S3 Replication is to maintain the same storage class as found on the Source Bucket, but you can override that in the replcation configuration (good if Destination Bucket will contain secondary data)
3rd: Ownership. Define ownership of objects in the Destination Bucket. Default is that dest buckets are owned by same entity that owns Source - This may not work if Source/Dest buckets are in different accounts (Dest bucket may not be able to read objects if objects are owned by Source).
4th: Replication Time Control (RTC): Adds guaranteed 15 minute SLA onto this process. This is used to make sure Source/Destination are in sync as closely as possible.
EXAM: 15 minute replication mentioned on exam? Then you know you need Replication Time Control (RTC)

### S3 Replication - Considerations:
EXAM:
- By Default, Replication is NOT retroactive and Versioning needs to be ON. Eg. If you enable Replication on a bucket that already has objects, those objects will not be replicated.
- Both Source AND Destination bucket need Versioning enabled
- You can use S3 Batch Replication to replicate pre-existing objects (since Replication is NOT retroactive by default)
- One-way replication: If you manually add objects to Destination Bucket, they will NOT be replicated to Source
-- Bi-directional replication is a recently added feature, but is an additional setting that needs to be configured
- Replication Capability: can handle unencrypted, SSE-S3 & SSE-KMS (with extra config), SSE-C
- Source Bucket Owner needs Permissions on the objects which will replicate
- Will NOT replicate: System events, Glacier, or Glacier Deep Archive
- NO DELETE markers are NOT replicated by Default. You can enable with DeleteMarkerReplication

### S3 Replication - Why Use It?
- For SRR (same region): For log aggregation, synchronize PROD and TEST accounts, resilience with strict sovereignty requirements (some data cannot leave specific regions)
- For CRR (cross region): Global resilience improvements, latency reduction

### S3 Replication - Resources:
- https://docs.aws.amazon.com/AmazonS3/latest/dev/replication.html
- https://aws.amazon.com/about-aws/whats-new/2019/11/amazon-s3-replication-time-control-for-predictable-replication-time-backed-by-sla/



## DEMO Cross-Region Replication of an S3 Static Website
Create 2 S3 buckets - one in N. Virginia, the other in N. California and configure Cross-Region Replication (CRR) between the two.

1. iamadmin account, N.VA region selected
2. Nav to S3 > Create Source Bucket > Create Bucket > name of source "sourcebucketta[random number]" > region us-east-1 > Create Bucket
3. Enable Static Website on Source > Properties Tab > Static website hosting: edit: Enable > type: static website > index doc "index.html"/error doc "index.html" > Save changes
4. Edit Source bucket Permissions for Public Access > Permissions tab > uncheck Block All Public Access > confirm > Save changes
5. To made Source Bucket public, add policy > Permissions tab > Bucket Policy "edit" > from lesson docs, paste JSON, replace Resource arn before the "/*" > Save
6. Create Destination Bucket & Set Permissions/Policy > Create Bucket > name of dest "destinationbucketta[random number]" > AWS Region: us-west-1 > uncheck Block All Public Access > Properties tab, Enable Static website hosting > Hosting type: static > index/error docs "index.html" > Save changes > Permissions tab, edit Bucket policy, paste JSON, update ARN, Save
7. Enable Cross-Region Replication (CRR): Source Bucket Management tab > Create replication rule > Enable Versioning > Replication rule name "staticwebsiteDR" > Status "Enabled" > Choose Rule Scope: "Apply to All objects in the bucket" > Destination: Browse S3, find destination bucket, enable versioning > IAM Role: dropdown "create new role" > Create replication rule > Replicate Existing Obejcts? No (as we have no pre-existing objects)
8. Clean Up: Empty/Delete Destination Bucket > Empty/Delete Source Bucket > IAM: locate role staring with "s3crr_role[...]"


# S3 PreSigned URLs
Presigned URL's are a feature of S3 which allows the system to generate a URL with access permissions encoded into it, for a specific bucket and object, valid for a certain time period
- PreSigned URLs can be used for downloads (GET) or Uploads (PUT)

Eg. S3 bucket with NO public access configured. Currently, a user would have to authenticate in IAM and be authorized to access resource. If you need to give an unauthenticated user access to the bucket, there are 3 [un-ideal] solutions 1) give mystery user AWS ID 2) give myst.user credentials 3) make object public.
- Better solution than all this is PreSigned URLs

NOTE: The PreSigned URL is used, the holder of the URL is interacting with S3 as the person who GENERATED the URL. In the example above, the mystery user would be iamadmin

EXAM: You can create a URL for an object that you have NO ACCESS TO. But, since you have no access, the PreSigned URL won't either. Not an applicable use case, but possible
EXAM: When using a PreSigned URL, the permissions are the same as the CURRENT permissions of Identity that generated the PreSigned URL. (So Access Denied could mean the generating ID never had access, or doesn't now)
EXAM: Don't generate PreSigned URLs using an IAM Role (temp credentials of a Role can expire before temp access to PreSigned URL)


 ## DEMO S3 Creating and using PreSigned URLs
 Create a bucket, upload an object and generate a presignedURL allowing access for any unauthenticated identities.

 1. Create bucket > name "animals4lifemedia[randomnumber]" > Create bucket
 2. Upload object > all5.jpv to new bucket
 3. Generate PreSigned ULR > Cloud Shell (terminal icon top right of AWS) > shell command "aws s3 presign s3://animals4lifemedia745675/all5.jpg --expires-in 180"
 - 180 is in seconds, 3 mins
 4. Clean up: Empty/delete bucket

 NOTE: Interesting Aspects...
 1. Try "aws s3 presign s3://animals4lifemedia745675/all5.jpg --expires-in 604,800"
 2. Nav to IAM > Users > iamadmin > Permissions: Add inline policy, copy JSON from lesson > save
 3. In CloudShell, try "aws s3 ls", you'll see Access Denied. This Explicit Deny overrules S3 permissions. Refresh PreSigned URL and see that access is now denied as the current permissions of iamadmin are restricted from S3
 4. iamadmin currently restricted from S3, but can still generated a PreSigned URL for it, even with no access.
 - You can also generate a PreSigned URL on a non-existent object
 - If you generate a PreSigned URL with an assumed Role, the URL will stop working with the temporary creds for the Role stop working
 - Can now create PreSigned URL from AWS Dashbord s3 > bucket > object > Object Actions dropdown "Share with a presigned URL"


## S3 Select and Glacier Select
EXAM: Understand this architecture
S3 and Glacier Select allow you to use a SQL-Like statements to retrieve partial objects from S3 and Glacier.
- Ways you can retrieve PARTS of objects rather than the entire object
-- Both S3 and Glacier super scalable. S3 can store objects up to 5TB and store infinite number of these objects. Often, you don't need to retrieve the entire 5TB object
- With S3 Select, filtering occurs on the service/the source (S3); The data delivered by S3 is pre-filtered WITHIN S3, making it quicker and reducing costs


## S3 Events
The Amazon S3 notification feature enables you to receive notifications when certain events happen in your bucket. To enable notifications, you must first add a notification configuration that identifies the events you want Amazon S3 to publish and the destinations where you want Amazon S3 to send the notifications. You store this configuration in the notification subresource that is associated with a bucket
- Notifications generated when events occur in a bucket
- Can be delivered to SNS Topics, SQS Queues and Lambda Functions
- Types of events: object Created (Put, Post, Copy, CompleteMultiPartUpload), object Delete (*, Delete, DeleteMarkerCreated), object Restore (Post (Initiated), Completed), Replication
- Events are generated by S3 Service, known as S3 Principal, so we also need to add Resource Policies to the destination services to allow S3 svc to interact
- Events are JSON objects
- S3 Event notifications are dated and limited features. Can also use EventBridge which supports more events and wider svc integration.
DEFAULT: use EventBridge as Default for S3 event notifications


## S3 Access Logs
Server access logging provides detailed records for the requests that are made to a bucket. Server access logs are useful for many applications. For example, access log information can be useful in security and access audits. It can also help you learn about your customer base and understand your Amazon S3 bill.
- You have a Source bucket where you want to track access. You have a Target bucket where you'll store tracked events. Can be enabled via Console UI or via PUT Bucket Logging
- S3 Logging managed by S3 Log Delivery Group
- Logs delivered as Log Files, consisting of Log Records
-- Each attribute within a record is space-delimited, and Records within in a file are newline-delimited
- Single Target bucket can be used for many Source buckets, separate logs using prefixes in Target bucket


## S3 Object Lock
You can use S3 Object Lock to store objects using a write-once-read-many (WORM) model. It can help you prevent objects from being deleted or overwritten for a fixed amount of time or indefinitely. You can use S3 Object Lock to meet regulatory requirements that require WORM storage, or add an extra layer of protection against object changes and deletion.
- WORM Model - wrote-once-read-many. Once Object Versions are created, they can't be overwritten or deleted (object versions are locked)
- User can enable on 'new' buckets, support ticket required for adding to existing bucket
- Object Lock required Versioning to be enabled. Once you create/enable Object Lock, you can't disable/delete OL or Versioning
- Can define Object Lock on objects, or a Bucket can have a default Object Lock Settings
- You can overlap the effects, Eg. Having a Legal Hold and Retention: Governance enabled on same object

Object Lock handles retention in TWO ways:
1. Retention Periods
2. Legal Holds
- Can have both, either, or none

### S3 Object Lock - Retention Period Methods - Retention Periods and Legal Holds
#### Retention Period: When you create lock, you specify retention period for DAYS / YEARS
Two Modes of Retention Period (EXAM):
- 1. Compliance Mode. Object version Can't be adjusted, deleted, overwritten for duration of period. Retention period duration can't be reduced. Retention Mode cannot be adjusted during period (EXAM: not even by Account Root User). This is the most strict Object Lock: Retention Period w/ Compliance Mode.
- 2. Governance Mode. Set a retention period, but special permissions can be granted allowing Lock Settings to be adjusted during the retention period
-- EXAM: s3:BypassGovernanceRetention in order to allow settings adjustment, along with header along with request "x-amz-bypass-governance-retention:true" (console ui default)

### S3 Object Lock - Lock Legal Hold
With Legal Hold, you don't set a Retention Period at all. Instead, for an Object Version, you set Legal Hold ON or OFF; binary. NO RETENTION.
- While Legal Hold flag is on, no deletes and no changes until Legal Hold is removed
-- s3:PutObjectLegalHold required to add or remove
- Legal Hold can be used to prevent accidental deletion of critical object versions. Or if you need to flag an object version for a legal case/project etc


## S3 Access Points
Amazon S3 Access Points, a feature of S3, simplifies managing data access at scale for applications using shared data sets on S3. Access points are unique hostnames that customers create to enforce distinct permissions and network controls for any request made through the access point.
- You can have 1 bucket with many access points, each access point can have different policies
- Access Points can be limited in terms of WHERE they can be accessed from. Eg. VPC or internet
- Every Access Point has its own endpoint address
- EXAM: Access Point created via Console or with this shell command:
-- aws s3control create-access-point --name [name] --account-id [account-id] --bucket [bucket-name]
- Access Points can be thought of as mini buckets or different views of the bucket
- Each Access Point has a unique DNS that would be given to the users using it
- Access Point Policies control permissions for access via Access Point and are functionally equivalent to a Bucket Policy. Access Point Policy can restruct identities to certain prefixes, tags, or actions based on need
- IMPORTANT: Any permissions defined on an Access Point need to also be defined on the Bucket Policy

### S3 Access Points Resource:
https://docs.aws.amazon.com/AmazonS3/latest/dev/creating-access-points.html#access-points-policies


## Demo S3 Multi-Region Access Points (MRAP)
Gain practical experience working with Multi-region access points

Amazon Simple Storage Service (S3) Multi-Region Access Points provide a global endpoint for routing Amazon S3 request traffic between AWS Regions. Each global endpoint routes Amazon S3 data request traffic from multiple sources, including traffic originating in Amazon Virtual Private Clouds (VPCs), from on-premises data centers over AWS PrivateLink, and from the public internet without building complex networking configurations with separate endpoints.

1. Create two buckets: S3 > Create bucket "multi-region-demo-sydney-[random-number]", region ap-southeast-2, Enable Bucket Versioning > Create > Create 2nd bucket "multi-region-demo-canada-[random-number]", region ca-central-1, Enable Bucket Versioning > Create
2. Create Multi-Region Access Point: S3 > Multi-Region Access Points > Create Multi-Region Access Points > name "reallyreallycriticalcatdata", add the 2 created buckets > Create (MRAP ARN arn:aws:s3::704310952405:accesspoint/m6s3xgw995fsb.mrap)
*NOTE: You CANNOT add or remove buckets to Multi-Region Access Point after it's cerated
3. Configure replication between the buckets: Access new MRAP > Replication & Failover tab > template:replicate among all specified buckets, Buckets: check both, Scope: Apply to all objects in bucket > Create replication rules
4. Test with CloudShell > Switch region to Tokyo > open CloudShell > enter "dd if=/dev/urandom of=test1.file bs=1M count=10" > copy ARN from MRAP dashboard, enter "aws s3 cp test1.file s3://[your-copied-mrap-arn]"
- Steps 3 and 4 generate a test file for upload, then uploads to the Sydney bucket. Replication of file to Canada bucket may take a few minutes
5. See remaining steps in the demo file in this repo
6. Clean up: s3 MRAP > select MRAP, delete > s3 buckets Empty and Delete

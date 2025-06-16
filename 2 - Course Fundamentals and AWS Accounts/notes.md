# AWS Accounts

- AWS Accounts are not only required to use AWS services, they are also one of the most important tools available to a solutions architect.

- Many students confuse accounts with users inside the accounts.
  - An organization ("org") may use one account or many thousands.
  - Bigger orgs generally use multiple AWS accounts.

- At a high level, AWS Account is a *container* for Identities (users) (which you log in with) and Resources (which you provision).

- When creating an AWS account you need to provide the following:
  - Unique email address (for EACH AWS account); this creates a special type of identity: the account root user
  - Payment method, generally a Credit Card (No need to spend: You can choose a Free tier and create a zero-spend budget)

- Root Users can only access their own accounts.
  - Initially, it is also the only user. Root user has full control over the one specific AWS account and any resources inside it.
  - **The root user CANNOT be restricted in any way.**
  - Protect the root username and password and don't use them unless necessary.

## Billing

- The Credit Card used to open account is the Account Payment Method.
- Any billable usage is charged to this CC.
- "Pay-as-you-go" platform.
- There is a free tier which we use in this course.

## Security

- You can create restricted identities/users using Identity and Access Management (IAM).
  - *Users*, *Groups*, and *Roles* can be created and given FULL or LIMITED permissions.
  - All of these ID's start off with NO access/permissions in the account.

- The IAM service is Account specific.
  - Cross-account permissions are a possibility as well, as we'll see later.

## Boundary of the Account

- AWS accounts are really good at containing what's inside the accounts; bad employee or bad actor, etc.
  - Having separate AWS accounts for different uses (DEV, PROD, etc), allows you to limit overall damage from a bad actor.

- By default, all access to an AWS account is denied except for the root user.

- [TIP]: Create new accounts for the course, don't use existing AWS accounts. The longer an AWS account exists, the more potential for configuration errors.

# [DEMO]: Creating the GENERAL AWS Account:

In this demo, we'll:

1. Create a General AWS account (*MANAGEMENT). This account's root user will be what we log in with (root user = account specific).
2. Add root user MFA for security.
3. Create a Budget to protect against unintended costs.
4. Create an IAM user, IAMADMIN. Give permissions. Then we'll use this ID for the course.

## 1. Create General AWS account

- https://aws.amazon.com/resources/create-account/ - Root user email address, AWS account name
  - TIP: use one email for multiple accounts with Gmail. AWS accounts should be viewed as disposable, create as many as you need. Create a new account for each course.
  - Example: email is catguy@gmail.com. You can use + sign in email address to create 'unique' email addresses. Ex: catguy+AWSAccount1@gmail.com, catguy+AWSAccount2@gmail.com, etc etc. This is called a Dynamic Alias.

- Choose free tier AWS account after providing all account setup info (unique email, CC, verification).
- Complete the prompted steps. Account will now be created.

### Account Tips

- [TIP]: It's always a good idea to update Alternate Contacts if you're using this account in the real world
- [TIP]: To allow IAM User and Role Access to Billing Information, click "Edit", check Activate IAM Access
  - If this IAM box isn't checked, even an IAM ID will full admin permissions wont't be able to access the billing console


## 2. Add root user MFA to secure the General AWS account

- In AWS Console > Account (top right) > Security Credentials to IAM console > locate "Assign MFA Device", follow steps.
- Recommend using Authy
- Log Out and test MFA login.
- By end of this course, you should have a total of 4 virtual MFA's configured in the same fashion (2 for each account -- 1 for root and 1 for admin).

### Multi-Factor Authentication (MFA) Explanation

- MFA makes things more secure with an additional One Time Password (OTP) besides the normal password

- Definition: Factors - Different pieces of evidence which prove identity

- 4 Commonly used Factors:
  - Knowledge (username / PW)
  - Possession (debit card, MFA device, MFA app, etc)
  - Inherent - Something you are: fingerprint, biometrics, etc.
  - Location - Physical location, which network (corp or wifi)

More factors means more security and harder to fake. Also means less convenient. We're trying to strike a balance between security and convenience.

## 3. Create a Budget to protect against unintended costs

- AWS Free Tier Benefits: https://aws.amazon.com/free/
  - Details allocations of free resources

- Create Cost Budgets: click "Budgets" > select Use a Template > select an appropriate option based on monthly spend budget (select Zero spend budget or Monthly Budget) > Budget Name, enter Email Recipients for alerts ([EMAIL]+trainingawsgeneral@gmail.com) > click "Create Budget"

- Budgets allow you to monitor spend and configure alerts when hitting spend targets

## Creating the Production Account

After creating General AWS account, we now need to create multiple AWS accounts and connect them with an AWS Organization.

### Create Production AWS Account retracing steps 1,2,3

Steps:
1. First, decide on email address to use [EMAIL]+trainingawsproduction1@gmail.com
2. Credit Card
3. Support Plan (Free Tier)
4. Secure AWS account with MFA to Root user of Production Account
5. Billing Preferences, check the boxes, adding email address for alerts
6. Create budget (Zero Spend Template or Monthly Budget)
7. Enable IAM User & Role Access to Billing (in Accounts)
8. Optional: Update alternate contacts in Account


## 4. Create an IAM user, IAMADMIN and give it permissions

Search "IAM" > IAM Dashboard > Create user "iamadmin"
- Create an IAM user (2nd radial option, 1st option to be covered later), check "Provide user access to the AWS Management Console - optional"
- Select second radial option, Create IAM User: Custom PW
- Give admin permissions "Attach policies directly" > check "AdministratorAccess"
- Create user
- Test login by visiting alias URL
- Confirm login with profile dropdown that will show account name
- Secure admin account with OTP > Security Credentials > Assign MFA
- Log out and test OTP

### [TIP]: To set alias (make it globablly unique)

- Within IAM Dashboard, find right-side AWS Account info, find "Account Alias", click "Create".
  - Must be a globally unique ID. For instance "ta-cantrill-training-aws-general"
  - Now URL is https://ta-cantrill-training-aws-general.signin.aws.amazon.com/console

### IAM (Identity and Access Management) Basics

- We first need to understand the identity situation. Accounts created have Root user will have full access. AWS account and Root user can be thought of as the same thing.

- Generally, you want to give access to other people in the companies, but with restricted access -- ONLY GIVE THEM PERMISSIONS REQUIRED TO DO A JOB, called **"Least Privileged Access".**

- [EXAM]: IAM is a GLOBALLY RESILIENT service, so any data is always secure across all AWS regions.

- Any IAM's in an account are completely separate from any other accounts. IAM as a service can do as much as the Root user if given all permissions, exception being some billing stuff (but that can be activated).

- Inside IAM, you can create multiple identities.

- IAM let's you create 3 types of identity objects:
  - Users: Humans or applications that need access to your AWS account. An application that does backups of the account may also be an IAM user.
  - Group: Collections of related users. All dev team users, finance, HR, etc.
  - Roles: Can be used by AWS Services or if you want to grant external access to your account. Used to grant access to services in your account to an uncertain number of entities. Eg. If you want all EC2 instances in your account to access the Simple Storage Service (S3), you can create a role which grants access and allow instances to use that role.

[TIP]: ROLES should be used when the number of entities needing access is *uncertain*.

### IAM Policy or Policy Document

- Objects or documents that can be used to Allow or Deny access to AWS services.
- Work only when attached to users, groups, or roles.

### High Level IAM

- IAM has 3 main jobs:
  1. **IDP (ID Provider)**: Manages identities (kind of like a CRUD for ids)
  2. **Authentication**: Authenticates identities (when anyone makes request to AWS, they're known as a Security Principle--they must prove *they are who they say they are*)
  3. **Authorization**: Allowed or Denied access to resources. This is based on Policies associated with the identity that is authenticated with.

### Other IAM Key Points

- No cost for IAM
- Global service / Global Resilience (copes with failure of large parts of the AWS system)
- IAM only controls what its identities can do. Allows or Denies identities *in account*
- No direct control on external accounts or users. It only controls local accounts or users
- Identity federation (to be covered later, but allows you to use Facebook, Google and others to login) and MFA


# [DEMO] Adding an IAM Admin to GENERAL ACCOUNT

We'll create an IAM account with full permissions to take the place of the Root user. Root users should generally only be used to create an account. Afterward, best practice is to utilize an IAM User rather than account root user. We'll use this IAM user to create new identities with less permissions (only enough for the user to do a certain task).

- Set up AIM ID with Admin permissions
  - Search "IAM" > IAM Dashboard

## Set alias

- Within IAM Dashboard, find right-side AWS Account info, find "Account Alias", click "Create".
  - Alias must be globally unique
  - Now URL is http://[account-alias].signin.aws.amazon.com/console
  - To sign on with IAM identities from now on, use this sign-in URL.

## Create IAM Admin ID

- IAM Dashboard > left sidebar "Users" > Create user "iamadmin"
- Give Access to Console
- Create an IAM user (2nd option, 1st option to be covered later)
- Custom PW
- Give admin permissions "Attach policies directly" > check "AdministratorAccess"
- Create user
- Test login
- Confirm login with profile dropdown that will show account name
- Secure admin account with OTP > Security Credentials > Assign MFA
- Log out and test OTP

# [DEMO] Adding an IAM Admin to PRODUCTION ACCOUNT
Same as previous section.

# IAM Access Keys
Access AWS via command line or APIs. This is done using IAM access keys.

- User/Password and Access Keys are long term credentials (they don't change or rotate regularly automatically)
- IAM user has 1 username and 1 PW (cannot have more than 1).
  - Password on IAM user is actually optional.
  - Credential Leak: If someone knows your username (public) and password (private)
- Access key actions: create, delete, made inactive, made active. Default: Active state
  - Unlike username/PW, IAM user can have TWO access keys

- Access keys are formed from two parts, which are provided by AWS when key is created:
  1) Access Key ID (akin to the username)
  2) Secret Access Key (akin to the password)

- Once the key is generated, *you CANNOT access the Secret Access Key part again* (note it down!)
  - If leaked/lost, you need to delete ID and Secret Access Key and recreate completely
  - If made inactive or deleted, CLI will stop working until re-active or new key is created

- Rotating Access Keys: delete old one make new one and replace old one

- [TIP]: Don't give the Root user any Access Keys
- [TIP]: IAM Roles DO NOT use access keys

# [DEMO] Creating Access Keys and Setting Up AWS CLI v2 Tools

Once logged in with Admin user:
IAM Dropdown > Security Credentials > scroll down to "Create Access Key", click > Command Line Interface (CLI) > check box at bottom > Next > Set Decription Tag > "Create Access Key"

Once Access Key is created, you can use Actions dropdown to Deactivate, Activate, and Delete. If you ever lose access to a key, you need to deactivate & delete it, then create a new one.

### Download AWS CLI v2
AWS CLI v2 (Windows) Installation - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html
AWS CLI v2 (macOS) Installation - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html (or just use `brew install awscli`)
AWS CLI v2 (Linux) Installation - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html

### Configure CLI
Configure a set of credentials which CLI will use to communicate with AWS. We will use the General IAMADMIN user for this one.

```bash
aws configure # configures the default profile for CLI (we'll used named)
aws configure --profile iamadmin-general # named profile for CLI
aws configure --profile iamadmin-production # named profile for CLI
```

Upon entering the above Command:
- AWS Access Key ID
- AWS Secret Key
- Default Region Name: us-east-1
- Default output format (press Enter with blank field for default)

- Test that this is successful with COMMAND 'aws s3 ls --profile [CLI profile name]'
  - 'aws s3 ls --profile iamadmin-general' will currently return a blank string as there are no s3 buckets.

#### Configure for Production

```bash
aws configure --profile iamadmin-production
aws s3 ls --profile iamadmin-production # to test
```

SECURITY REMINDER: Never share your SECRET KEY. If leaked, delete and create new set of keys and re-configure in CLI

[TIP]: If after you Configure CLI with credentials, you can delete the credential files (CSVs)

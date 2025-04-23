# Public Introduction & Info

## Course Resources

- GitHub Repository: https://github.com/acantril/aws-sa-associate-saac03
- Discord: discord.gg/tsd

## Site Tools & Features

Products page, Discord channel, free GitHub repo: "Labs" link, mini projects in the GH repo, "More" tab for tech support

# AWS Exams

## How to get started

- Don't start with the Cloud Practitioner Foundational Cert, start with Associate level SAA (Solutions Architect Associate). The content overlaps.
- After you finish SA Associate do the Role Based certs, then Specialty certs later. Get SA Professional prior to Specialties certs
- In Associate level, start with:
  1. SAA
  2. Then Developer Associate
  3. Then SysOps Associate (Hardest exam of the 3)
- In Professional level, tests are way harder. Longer questions, more questions:
  1. Professional SA
  2. DevOps Pro

# Scenario - Animals4life

- Animal rescue/awareness Org.
- Global company, 100 staff, 100 remote staff, etc etc. Small data-center is old, and data should be migrated out.
- Badly implemented AWS trial in SYD Region. Isolated Azure/GCP Pilots. All staff uses this infrastructure. Company runs lean but will get new tech if it helps business.
- Several networks:
  - On Premise: 192.168.10.0/24, Class C
  - AWS Pilot: 10.0.0.0/16, Class B
  - Azure Pilot: 172.31.0.0/16, Class B
- Major offices: NY, Seattle, London all use Head Office services for data
- Field workers on laptop, 3g/4g/satellite for email/files, chat/planning, research data

## Problems:

- Hardware failing, datacenter decommissioned in 18months, so business needs to migrate out
- New investment needed, not sure if on premise, azure, aws, etc.
- Previous AWS/Azure attempts didn't pan out.
- Distance to datacenter is sub-optimal for much of company.
- Lean/appropriately sized, but trouble with peak traffic.
- IT team sucks at cloud.

## Ideal Outcomes:

- Fast performance for all field workers
- Able to deploy into new regions quickly when required
- Low cost and scalable base infrastructure, cost close to 0 as possible while meeting requirements
- Agility - new marketing campaigns, social and progressive applications (IOT, Big Data, etc). Wants to make use of emerging technologies
- Automation - low base staffing costs

- TIP: Pay attention to which areas need more information, so you can ask the right questions.

---
tags:
  - aws
  - certification
  - roadmap
  - cloud
source: https://career.softserveinc.com/en-us/stories/complete-aws-certification-guide
created: 2026-04-14
---

# AWS Certification Roadmap

A cleaned-up, structured guide based on SoftServe's article: [A Journey Through All AWS Certifications](https://career.softserveinc.com/en-us/stories/complete-aws-certification-guide).

> [!success] Current plan
> First target: [[AWS Certifications/02 - AWS Certified AI Practitioner]]
>
> Current recommended sequence:
> - [[AWS Certifications/02 - AWS Certified AI Practitioner]]
> - [[AWS Certifications/07 - AWS Certified Machine Learning Engineer - Associate]]
> - [[AWS Certifications/12 - AWS Certified Generative AI Developer - Professional]]
> - [[AWS Certifications/03 - AWS Certified Solutions Architect - Associate]]
>
> Execution note: use [[AWS AI Certification Study Plan]] as the active plan.

> [!tip] New linked note system
> Start with [[AWS Certification Hub]] for the visual roadmap and quick navigation.
>
> Individual certification notes live in:
> - [[AWS Certifications/01 - AWS Certified Cloud Practitioner]]
> - [[AWS Certifications/02 - AWS Certified AI Practitioner]]
> - [[AWS Certifications/03 - AWS Certified Solutions Architect - Associate]]
> - [[AWS Certifications/04 - AWS Certified Developer - Associate]]
> - [[AWS Certifications/05 - AWS Certified CloudOps Engineer - Associate]]
> - [[AWS Certifications/06 - AWS Certified Data Engineer - Associate]]
> - [[AWS Certifications/07 - AWS Certified Machine Learning Engineer - Associate]]
> - [[AWS Certifications/08 - AWS Certified Solutions Architect - Professional]]
> - [[AWS Certifications/09 - AWS Certified DevOps Engineer - Professional]]
> - [[AWS Certifications/10 - AWS Certified Security - Specialty]]
> - [[AWS Certifications/11 - AWS Certified Advanced Networking - Specialty]]
> - [[AWS Certifications/12 - AWS Certified Generative AI Developer - Professional]]

## What this guide is for

Use this note to:
- understand the AWS certification landscape
- choose the right path based on your role
- know what each exam tests
- plan study order and prep strategy

---

## AWS certification map

### Foundational
- **AWS Certified Cloud Practitioner**
- **AWS Certified AI Practitioner**

### Associate
- **AWS Certified Solutions Architect - Associate**
- **AWS Certified Developer - Associate**
- **AWS Certified CloudOps Engineer - Associate**
- **AWS Certified Data Engineer - Associate**
- **AWS Certified Machine Learning Engineer - Associate**

### Professional
- **AWS Certified Solutions Architect - Professional**
- **AWS Certified DevOps Engineer - Professional**
- **AWS Certified Generative AI Developer - Professional**

### Specialty
- **AWS Certified Security - Specialty**
- **AWS Certified Advanced Networking - Specialty**

---

## Best starting paths

### 1. General AWS / architecture path
1. Cloud Practitioner
2. Solutions Architect Associate
3. Solutions Architect Professional
4. Security Specialty or Advanced Networking Specialty

### 2. Developer / DevOps path
1. Cloud Practitioner
2. Developer Associate
3. CloudOps Engineer Associate
4. DevOps Engineer Professional

### 3. Data path
1. Cloud Practitioner
2. Data Engineer Associate
3. Solutions Architect Associate (optional but useful)

### 4. AI / ML / GenAI path (primary for now)
1. AI Practitioner
2. Machine Learning Engineer Associate
3. Generative AI Developer Professional
4. Solutions Architect Associate (strong complement)

---

## Certification summary table

| Certification | Level | Questions | Time | Passing | Main focus |
|---|---:|---:|---:|---:|---|
| Cloud Practitioner | Foundational | 65 | 90 min | 700 | AWS basics, pricing, core services, cloud concepts |
| AI Practitioner | Foundational | 65 | 90 min | 700 | AI/ML/GenAI concepts and AWS AI services |
| Solutions Architect Associate | Associate | 65 | 130 min | 720 | System design, service selection, security, scale |
| Developer Associate | Associate | 65 | 130 min | 720 | App development, deployment, CI/CD, AWS dev services |
| CloudOps Engineer Associate | Associate | 65 | 130 min | 720 | Operations, monitoring, troubleshooting, infra management |
| Data Engineer Associate | Associate | 65 | 130 min | 720 | Pipelines, analytics, warehouses, lakes, data services |
| Machine Learning Engineer Associate | Associate | 65 | 130 min | 720 | SageMaker-heavy ML workflows, deployment, model ops |
| Solutions Architect Professional | Professional | 75 | 180 min | 750 | Complex architectures, trade-offs, optimization |
| DevOps Engineer Professional | Professional | 75 | 180 min* | 750 | Deep CI/CD, automation, containers, reliability |
| Generative AI Developer Professional | Professional | 75 | 130 min | 750 | Bedrock, SageMaker, RAG, prompt engineering, GenAI systems |
| Security Specialty | Specialty | 65 | 170 min | 750 | IAM, KMS, logging, detection, secure architecture |
| Advanced Networking Specialty | Specialty | 65 | 170 min | 750 | Hybrid networking, routing, VPC, DX, TGW, network design |

> *The article stresses DevOps Pro as a long, difficult exam and frames it like the pro-tier experience. Verify the live exam guide before scheduling.

---

## Exam-by-exam breakdown

## 1) AWS Certified Cloud Practitioner
**Who it is for:** Beginners, non-cloud engineers, people needing AWS baseline literacy.

**Focus areas**
- cloud concepts
- shared responsibility model
- core AWS services
- pricing and billing basics
- basic architecture understanding

**Prep advice**
- learn what each major service does
- focus on breadth over depth
- get comfortable distinguishing similar services at a high level

**Good next step**
- Solutions Architect Associate

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-cloud-practitioner/)

---

## 2) AWS Certified AI Practitioner
**Who it is for:** People entering AI on AWS, especially if you need foundations before ML or GenAI.

**Focus areas**
- AI, ML, and GenAI basics
- Bedrock, Amazon Q, SageMaker
- prompt engineering
- governance, compliance, responsible AI
- supervised vs unsupervised learning, RLHF, bias/variance

**Prep advice**
- do not treat it like an easy “intro cert”
- study AI concepts, not just AWS product names
- understand common GenAI/ML terminology well

**Good next step**
- Machine Learning Engineer Associate or Generative AI Developer Professional

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-ai-practitioner/)

---

## 3) AWS Certified Solutions Architect - Associate
**Who it is for:** Engineers who design or reason about AWS systems.

**Focus areas**
- architecture best practices
- secure and reliable systems
- storage, compute, networking, and service trade-offs
- picking the most cost-effective and scalable solution

**Prep advice**
- go deeper than Cloud Practitioner
- do hands-on labs with core services
- practice choosing the *best* option, not just a valid option

**Good next step**
- Solutions Architect Professional

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

---

## 4) AWS Certified Developer - Associate
**Who it is for:** App developers building and deploying on AWS.

**Focus areas**
- application development on AWS
- CodeBuild, CodePipeline, CodeDeploy, Elastic Beanstalk
- deployment strategies
- service integrations for app delivery

**Prep advice**
- build a sample CI/CD pipeline end to end
- understand deployment modes and trade-offs
- go deeper on developer tooling than on broad architecture

**Good next step**
- DevOps Engineer Professional

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-developer-associate/)

---

## 5) AWS Certified CloudOps Engineer - Associate
**Who it is for:** Ops, SRE, infra, and platform-minded engineers.

**Focus areas**
- monitoring and observability
- CloudWatch, CloudTrail, CloudFormation
- VPC, NACLs, security groups
- troubleshooting and operational excellence

**Prep advice**
- hands-on is mandatory here
- practice alarms, logging, scaling, health checks, IAM, VPC setup
- know how to diagnose failures, not just deploy resources

**Good next step**
- DevOps Engineer Professional

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-cloudops-engineer-associate/)

---

## 6) AWS Certified Data Engineer - Associate
**Who it is for:** Data engineers and analytics engineers working with AWS pipelines.

**Focus areas**
- Glue, Redshift, S3, Athena, Kinesis, MSK
- ETL / ELT
- OLTP vs OLAP
- data warehouse vs data lake
- cost/performance trade-offs in analytics systems

**Prep advice**
- know the core data services cold
- learn data architecture patterns, not just service definitions
- understand when to use warehouse, lake, streaming, or event-driven components

**Good next step**
- deeper analytics specialization or broader architecture certs

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-data-engineer-associate/)

---

## 7) AWS Certified Machine Learning Engineer - Associate
**Who it is for:** Engineers implementing ML systems on AWS.

**Focus areas**
- SageMaker workflows
- data prep and feature engineering
- training, tuning, and deployment
- model versioning, drift, batch vs endpoint inference
- ML solution architecture on AWS

**Prep advice**
- treat this as a SageMaker-heavy exam
- know algorithms, tuning, deployment, and MLOps basics
- be comfortable with surrounding AWS services like IAM, S3, Glue, Athena, Lambda

**Good next step**
- Generative AI Developer Professional

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-machine-learning-engineer-associate/)

---

## 8) AWS Certified Solutions Architect - Professional
**Who it is for:** Experienced architects designing complex AWS systems.

**Focus areas**
- advanced architecture design
- performance, resilience, security, migration, cost optimization
- nuanced service selection under constraints
- interpreting difficult scenario-based questions

**Prep advice**
- expect harder wording and deeper trade-offs
- watch for keywords like managed, cost-effective, low latency, secure, minimal operations
- flag long questions and return if needed
- optimize time management aggressively

**Good next step**
- Security Specialty or Advanced Networking Specialty

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-solutions-architect-professional/)

---

## 9) AWS Certified DevOps Engineer - Professional
**Who it is for:** Senior DevOps / platform engineers working with automation-heavy AWS environments.

**Focus areas**
- CI/CD
- IaC
- automation
- monitoring
- ECS, EKS, ECR
- event-driven operations and secure delivery pipelines

**Prep advice**
- combine deep knowledge from Developer Associate and CloudOps Associate
- pay attention to keyword traps like real-time, near real-time, private, low latency, automatically
- if the question says avoid infra management, serverless is often the clue

**Good next step**
- Security Specialty

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-devops-engineer-professional/)

---

## 10) AWS Certified Security - Specialty
**Who it is for:** Security engineers or cloud engineers specializing in AWS security.

**Focus areas**
- IAM
- KMS
- CloudTrail
- CloudWatch
- Security Hub
- Inspector
- WAF and Shield
- encryption, audit, compliance, secure architecture

**Prep advice**
- depth matters more than breadth
- when in doubt, the most secure valid answer often wins
- know secure patterns for both data and network paths

**Good next step**
- pair with Solutions Architect Professional for strong cloud security credibility

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-security-specialty/)

---

## 11) AWS Certified Advanced Networking - Specialty
**Who it is for:** Engineers working on hybrid cloud, enterprise networking, or deep VPC connectivity.

**Focus areas**
- VPC architecture
- Direct Connect
- Transit Gateway
- VPN
- Route 53
- hybrid connectivity
- routing and network optimization

**Prep advice**
- this is one of the hardest AWS exams
- practice hybrid scenarios and routing decisions
- read carefully, because one word can change the answer

**Good next step**
- strong complement to architect or platform roles

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-advanced-networking-specialty/)

---

## 12) AWS Certified Generative AI Developer - Professional
**Who it is for:** Engineers building production GenAI systems on AWS.

**Focus areas**
- Bedrock
- SageMaker
- prompt engineering
- vector embeddings
- RAG
- model evaluation
- governance and responsible AI
- cost, latency, scale, safety trade-offs

**Prep advice**
- build at least one real GenAI app on AWS
- understand end-to-end solution design, not isolated services
- study model safety, hallucination mitigation, privacy, and moderation

**Good next step**
- combine with architect knowledge for strong AI platform credibility

**Official links**
- [Certification page](https://aws.amazon.com/certification/certified-generative-ai-developer-professional/)

---

## Cross-cutting prep strategy

### What matters for almost every AWS exam
- track progress visibly
- celebrate small milestones
- do hands-on labs, not just videos
- practice time management
- use mock exams seriously
- flag hard questions and return later
- read carefully, because AWS questions often hinge on one keyword

### Question-solving tactics from the article
- read the actual question carefully, not just the scenario
- sometimes read from the end first to find what is really being asked
- read answer options early to eliminate obvious mismatches
- group similar options and compare the one-word difference

---

## Recommended study order for a senior engineer

If your goal is broad cloud credibility:
1. Cloud Practitioner
2. Solutions Architect Associate
3. Developer Associate or CloudOps Associate
4. Solutions Architect Professional
5. Security Specialty

If your goal is platform / DevOps depth:
1. Cloud Practitioner
2. Developer Associate
3. CloudOps Associate
4. DevOps Professional
5. Security Specialty

If your goal is AI on AWS:
1. AI Practitioner
2. Machine Learning Engineer Associate
3. Generative AI Developer Professional
4. Solutions Architect Associate

---

## My take

If you only do **one** serious AWS cert first, make it **Solutions Architect Associate**.

It gives the best base for almost every other path. Cloud Practitioner is useful if you want a gentler ramp or need formal fundamentals, but SAA is where the real leverage starts.

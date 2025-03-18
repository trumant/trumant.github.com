---
layout: post
title:  "Implementing the NIST 800-53 Awareness & Training control family: a few easy buttons"
categories:
  - aws
  - security
  - compliance
  - fedramp
permalink: "/nist-800-53-awareness-and-training-control-family-easy-buttons"
---
![NIST National Institute of Standards and Technology](/assets/nist.png)

## So you need to implement the NIST 800-53 Awareness and Training (AT) control family

These controls have us:
 * recognize that every system user has a role to play in security and compliance
 * be thoughtful about which members of the organization require training and how it is delivered and maintained
 * keep records of our training activities
 * learn from organization-specific risk and incident history and incorporate those learnings into our training activities

Refer to the [NIST control language and assessment procedure details](https://csrc.nist.gov/projects/cprt/catalog#/cprt/framework/version/SP_800_53_5_1_1/home?element=AT) if you need all the gory details.

While you can spend your way to a successful awareness and training program, if you want to be scrappy and rely on free and open source resources, I've got a few easy buttons for you to consider in your compliance journey.

## AT-1 Policy & Procedures

It wouldn't be compliance without policy documentation!

My engineering bias is to automate the management of policy and procedure documentation and rely on Github for storing the source documents. This approach can serve us well in compliance where we always need to provide evidence of policy change control, version history and strong access management when granting permission to make policy changes.

The [security-policy-templates project](https://github.com/JupiterOne/security-policy-templates) from [JupiterOne](https://www.jupiterone.com) is an open source solution to this problem that can be easily adopted and extended. From their README.md:

> A set of foundational but comprehensive policies, standards and procedures designed for cloud-native technology organizations. The policy package covers the requirements and controls for most compliance frameworks and best practices, in a lightweight approach.

Put another way, you can fork this repository, adapt the policy language template fragments to your business and compliance needs and then maintain and version those templates and some configuration data that serves as template inputs in Github. When you need to produce your documentation, you run `psp build -t ./templates -c path/to/your/config.json` or setup a document publishing CI job and boom you've got a collection of Markdown documentation containing all of your policies.

Here is an example of the index created that links to all of the relevant documents:

![jupiterone/security-policy-templates Markdown index screenshot](/assets/policy_docs.png)

This project has great potential to be an easy button for your organization from my engineering-centric perspective. I would consider it a stop gap solution for most organizations, providing quick time to compliance while deferring investment in a commercial solution. However, keep in mind your policy and procedure stakeholders: Legal, Privacy, CISO, etc and whether or not they are going to be comfortable authoring and reviewing policy changes using Markdown and Github, even if only for a short while.

## AT-2 Literacy Training & Awareness

Every member of the organization operating within the compliance boundary needs a baseline level of security literacy and awareness training. You likely already have such training in place as part of your onboarding and annual training cycles, as thankfully most modern organizations have now been doing this for going on a decade. However, if you do not yet have a solution in place or want to consider a free solution the Linux Foundation comes to the rescue.

Linux Foundation's [Cybersecurity Essentials course](https://training.linuxfoundation.org/training/cybersecurity-essentials-lfc108/) is available for free and should take no more than an hour to complete. Completion of the course can be tracked/recorded in the Linux Foundation profile of each user taking the test as a "digital badge" is issued. Badges specify the date they were issued and this data can contribute to your AT-4 requirements to record testing activities.

You can see how the badges are displayed in both my own Linux Foundation [public profile](https://openprofile.dev/profile/trumant) and Credly [public profile](https://www.credly.com/users/travis-truman)

![LFC108: Cybersecurity Essentials training from the Linux Foundation](/assets/cybersecurity_essentials.png)

This combination of basic training, offered for free and including recording could be a real easy button if you aren't yet ready to invest in licensing a commercial solution.

This is a very effective training option for those members of your organization who interact with your business systems, rather than those who are developing software or administering critical IT or security systems. I cover more deeply technical training options in the role-based training section below.

## AT-3 Role-based Training

Those specialized roles in your organization that operate, secure and make changes to the systems within the compliance boundary require a tailored training approach. A typical training policy would require initial training as part of organizational onboarding and then subsequent training every year thereafter. I strongly recommend these free resources from the Linux Foundation, AWS and NIST that are suitable for engineering-specific roles in most organizations.

### All the trainings!

| Role | Relevant training | Time investment |
| ---- | ----------------- | --------------- |
| Frontend, Backend or Fullstack Software Engineers | [Understanding the OWASP® Top 10 Security Threats from the Linux Foundation](https://training.linuxfoundation.org/training/owasp-top-ten-security-threats-skf100/) | 12 hours |
| Frontend, Backend or Fullstack Software Engineers | [Developing Secure Software from the Linux Foundation](https://training.linuxfoundation.org/training/developing-secure-software-lfd121/) | 16-20 hours |
| Frontend, Backend or Fullstack Software Engineers | [AWS Threat Modeling for Builders workshop](https://explore.skillbuilder.aws/learn/courses/13274/threat-modeling-for-builders-workshop) | 6 hours |
| Software Engineers, DevOps, Security and any other engineers with access to AWS | [Security for Developers hands-on workshop](https://catalog.workshops.aws/sec4devs/en-US) | 3-4 hours |
| Security and DevOps engineers with access to AWS | [AWS Skillbuilder Security Learning Plan](https://explore.skillbuilder.aws/learn/learning-plans/91/security-learning-plan) | 2-3 hours |
| Security and DevOps engineers with access to AWS | [AWS Skillbuilder US Federal AWS Cloud & NIST Cybersecurity Framework Learning Path](https://explore.skillbuilder.aws/learn/learning-plans/2061/us-federal-aws-cloud-nist-cybersecurity-framework-learning-path) | 7-8 hours |
| Security and DevOps engineers implementing compliance controls | [NIST's Security and Privacy Controls Introductory Course](https://csrc.nist.gov/Projects/risk-management/rmf-courses) | 3-4 hours |
| Engineering leaders | [Security for Software Development Managers](https://training.linuxfoundation.org/training/security-for-software-development-managers-lfd125/) | 1-2 hours |
| Engineering leaders | [Introduction to DevSecOps for Managers](https://trainingportal.linuxfoundation.org/courses/introduction-to-devsecops-for-managers-lfs180/) | 20 hours |

<br>
While it can be tempting to only invest in training individual contributors, focusing training first for your engineering leaders will be a huge easy button. If you are rolling out a new training program, win over and train the leaders first with [Security for Software Development Managers](https://training.linuxfoundation.org/training/security-for-software-development-managers-lfd125/) and solicit their feedback on how to best rollout training to their ICs. The more these leaders understand security and compliance, the more likely they are to prioritize and invest and get higher quality outcomes from those investments within their domain. The more they are engaged and have had feedback on IC training rollout, the more likely that training is to be prioritized and completed.

All of these free options for role-specific training should be an easy button to reduce spend on training or get people trained more quickly without waiting for a procurement process. I hope you find them useful.

## AT-4 Training Records

All of the training options I shared above offer records course completion and completion date. If you can marry those 2 pieces of data up with the name and role of the person who took the training and supply that consistently and easily then you have evidence of meeting the control.

You'll need to store this evidence for your organizationally-defined retention period.

While a Google or Excel sheet may suffice in the short term, you'll ultimately want more robust records management capabilities if you are training a large organization. If you've found success with either the AWS Skillbuilder or Linux Foundation training options I provided above, both organizations provide commercial training options that will provide substantial improvements in record keeping.

## Conclusions

I hope some of the resources I've provided are helpful as you and your organization work to meet this control family. More importantly, I hope meeting the control strengthens a culture of continuous security learning in your organization. There is always more to learn and investments here will pay long term risk reduction dividends.

<a title="NBC, Public domain, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:The_More_You_Know_2022_logo.png"><img width="512" alt="The More You Know 2022 logo" src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/89/The_More_You_Know_2022_logo.png/512px-The_More_You_Know_2022_logo.png?20230922215825"></a>
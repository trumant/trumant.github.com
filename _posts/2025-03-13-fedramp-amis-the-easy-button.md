---
layout: post
title:  "FedRAMP compliant AMIs in AWS: a few easy buttons"
categories:
  - aws
  - security
  - compliance
  - fedramp
permalink: "/fedramp-compliant-amis-in-aws.html"
---
## So you want to be FedRAMP compliant AND directly manage EC2 instances

If you plan to or currently manage and operate EC2 instances in your AWS infrastructure supporting a FedRAMP or StateRAMP/GovRAMP solution, you have a dizzying number of controls to keep in mind and requirements to meet. While I can't promise an answer to every challenge you might face, I hope the tips below prove helpful and perhaps help you find a few easy buttons in your compliance journey.

### #1 - You can't control what you can't see and report on

Before diving into implementation ensure you have an efficient, repeatable and high quality views into the EC2 fleet that lives within your compliance boundary. This likely includes more than a single account and region so don't rely on the AWS console. Consider that your technical audience probably needs one level of detail and your compliance partners want a completely different level of detail. This is where cloud security posture management and asset inventory solutions shine and provide you with the first easy button, so hopefully you already have one in place that provides the level of visibility into EC2 that will be required. If not, find the gaps and prioritize closing them now.

### #2 - Reduce your reliance on directly managed EC2 instances

This should go without saying, but your compliance journey will be easier the more you can reduce the degree of compliance responsibility you own vs the responsibility delegated to AWS. By moving application workloads and datastores off of directly managed EC2 instances and into AWS services like EKS Fargate, ECS Fargate, Lambda, Step Functions, etc or adopting AWS-managed data stores like Aurora, DynamoDB and ElastiCache you've driven a substantial reduction in the surface area of host and operating system level compliance requirements you need to deliver.

Another common way to reduce your reliance on EC2 instances in cases where you must maintain some EC2 fleet is to replace any usage of EC2 jumphosts/bastion hosts with [SSM Session Manager](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-with-systems-manager-session-manager.html) and/or [EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-with-ec2-instance-connect-endpoint.html)

Fewer EC2 instances to manage is a great easy button to exploit, but of course the degree to which this is "easy" varies wildly based on workloads and organizational capabilities.

### #3 - Reduce the number of operating systems you run

The more consistency you can drive in both operating system distributions (Amazon Linux, RHEL, Ubuntu, etc), but also version usage across EC2, the smaller the surface area you will need to control and the higher reuse you can expect in your solutions. You are likely to be able to drive cost efficiencies through converging on fewer OSes, but keep in mind that some vendor distributions, example [Ubuntu](https://ubuntu.com/security/certifications/docs/fips), may want more money to provide the FIPS 140-3 support required.

If you can converge to a single Linux distribution and only several versions of that distribution you've found your third easy button.

### #4 - Build your compliant AMI factory with Image Builder

Because all EC2 instances are launched from an Amazon Machine Image (AMI) that is made available within the AWS account and region, we can employ a number of strategies and automated mechanisms supporting the production and usage of these AMIs within our infrastructure to meet our compliance needs. By default within an AWS account, the AMIs available are those produced by AWS and the community, those provided through vendors in AWS Marketplace and those we produce ourselves. Your AMI factory should automate the creation of hardened AMIs based on AWS or vendor provided base AMIs. Your AMI factory should trigger the creation of new AMIs whenever the base image is updated, whenever a new version of the DISA STIG hardening component is released for your OS and on a regular cadence like every 30 days in order to ensure that you are able to keep up with updates and patches across your fleet.

If you are running Windows, you can take advantage of the [AWS-managed STIG-hardened AMIs](https://docs.aws.amazon.com/ec2/latest/windows-ami-reference/ami-windows-stig.html). If you are running Linux distributions, expect to have to build some of your own hardening pipelines based off of AWS or community-provided base AMIs.

Two of the most popular tools for producing AMIs with automation are [Packer](https://www.packer.io) and [EC2 Image Builder](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html). While I've been a decade+ long user of Packer, when it comes to solving compliance challenges in AWS, EC2 Image Builder provides several easy buttons I would encourage you to exploit.

The first is the [AWS-managed automation that applies STIG hardening](https://docs.aws.amazon.com/imagebuilder/latest/userguide/ib-stig.html#linux-os-stig) to the most popular operating systems you are likely to use in your AMIs. This and the related [SCAP compliance validator component](https://docs.aws.amazon.com/imagebuilder/latest/userguide/ib-stig.html#scap-compliance) allow you to quickly apply the appropriate OS-level hardening to your AMIs and collect evidence of this hardening that can be provided to demonstrate compliance.

The second huge benefit that Image Builder offers over Packer is that it provides the reusable automation pipeline and framework mechanisms that make it easy to operate 10s or 100s of pipelines whereas Packer users are required to create and maintain bespoke pipelines for this purpose in their CI platform of choice. Image Builder allows you to separate your automation tasks into build vs test phase tasks, provides lifecycle management of images, makes distributing AMIs across multiple accounts and regions easy and integrates easily with other AWS services like Inspector, SNS and EventBridge that can be used to add security scanning and automated notification of new images with minimal effort.

Finally, using Image Builder allows you to tightly segment and control your CI pipeline infrastructure that is security-specific from more general purpose or application-specific CI platforms. Best practice if you are using AWS Organizations recommended account responsibilities model would be to place your Image Builder pipelines in your security tooling account.

### #5 - Buy your compliant AMIs

For some organizations, it may be advantageous to rely on vendor provided compliant AMIs. The [Center for Internet Security](https://www.cisecurity.org/cis-hardened-images/amazon) is one popular source for general purpose Linux AMIs. Consider the relative ROI of the purchase vs the investment in building your own AMI factory.

You must also understand to what degree you rely on vendor provided AMIs today within your compliance boundary. Reach out to those vendors and seek their guidance on meeting your compliance requirements with their images. Raise feature requests with them to provide more transparency into how they produce those images, what guarantees they provide and what benchmarks have been applied.

### #6 - Govern AMI usage across your compliance boundary

Once you've determined which AMIs implement your compliance requirements, you need to ensure that EC2 instances launched within your compliance boundary are using only those AMIs. All AMIs you produce can be [shared](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/share-amis-with-organizations-and-OUs.html) across your entire AWS Organization or to all accounts within an OU.

The [Allowed AMIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-allowed-amis.html) feature in EC2 is the last easy button. You can configure the criteria that define which AMIs are available for use within each account and region. The easy button criteria is to specify that only images produced by AWS and by the AWS account that houses our AMI factory are made available.

### #7 - Treat your EC2 instances like cattle, not pets

The final easy button is to treat your EC2 instances like cattle, not like pets. If we embrace this and other DevOps philosophies with respect to how we manage our EC2 instances every instance is relatively ephemeral with instances being terminated regularly due to autoscaling and lifecycle management and new instances using the latest version of the compliant AMIs are regularly being launched using autoscaling or other automation. This degree of constant change in an EC2 environment will naturally tend to ensure newly produced AMIs are consumed, but do not depend on it to ensure adoption.

You should consider setting a maximum age for any EC2 instance within your compliance boundary and a policy that any instance reaching that age is terminated and replaced with a new one using the latest AMI. This gives you assurances that no instance within your boundary has gone more than N days without the latest hardening configurations and patches.
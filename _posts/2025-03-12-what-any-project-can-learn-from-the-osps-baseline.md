---
layout: post
title:  "What any project can learn from the Open Source Project Security Baseline"
categories:
  - opensource
  - security
permalink: "/what-any-project-can-learn-from-the-osps-baseline.html"
---
## What is the baseline?

Even if you've been involved in open source projects and communities for some time, you may not
be familiar with the [Open Source Security Foundation (OpenSSF)](https://openssf.org/about/). Their mission is:

> make it easier to sustainably secure the development, maintenance, and consumption of the open source software (OSS) we all depend on

![Open Source Security Foundation (OpenSSF)](/assets/openssf_logo.png)

They support this mission through an abundance of [projects](https://openssf.org/projects/) like Sigstore and SLSA and recently announced the initial release of the [Open Source Project Security Baseline](https://baseline.openssf.org/versions/2025-02-25).

> The OSPS Baseline offers a tiered framework of security practices that evolve with project maturity. It compiles existing guidance from OpenSSF and other expert groups, outlining tasks, processes, artifacts, and configurations that enhance software development and consumption security. By adhering to the Baseline, developers can lay a foundation that supports compliance with global cybersecurity regulations, such as the EU Cyber Resilience Act (CRA) and U.S. National Institute of Standards and Technology (NIST) Secure Software Development Framework (SSDF).

## What can we learn from it?

I'm going to cherry-pick a few of the baseline's controls for commentary, but I strongly encourage you to read through all of the controls [in detail](https://baseline.openssf.org/versions/2025-02-25#controls-overview) and the [FAQ](https://baseline.openssf.org/faq).

### Input validation and injection defense

The first control I want to highlight is [OSPS-BR-01](https://baseline.openssf.org/versions/2025-02-25#osps-br-01---the-projects-build-and-release-pipelines-must-not-permit-untrusted-input-that-allows-access-to-privileged-resources) **_- The project's build and release pipelines MUST NOT permit untrusted input that allows access to privileged resources_**.

While this sounds great in principal, it's a bit vague. Reading further, the specific requirement for the control is:

> When a CI/CD pipeline accepts an input parameter, that parameter MUST be sanitized and validated prior to use in the pipeline.

Here we have immediately actionable guidance for any pipeline implementation. Imagine your project has a release pipeline that produces a Docker image using the contents of a Github repo, signs that image and publishes it as an "official release" in your container image repository.

If that pipeline accepts a git tag as input, the pipeline MUST validate the input by comparing it to an allowlist or acceptable patterns to ensure that only release tags are used. Allowing any tag would allow anyone with permissions to create tags in the repository to abuse the pipeline and distribute a release from code that they control and may not have been subject to standard release change control, security and compliance reviews.

In our example above, we can implement the control requirement by creating a Github repository tag ruleset that restricts the creation of tags matching `releases/**` to privileged identities and modifying the pipeline to validate the tag input provided matches `releases/**`.

A common gotcha I've observed in projects that define and manage their pipelines "as code" is the failure to recognize that this then creates a situation where proposed changes (a pull request/merge request) to the pipeline definition itself becomes untrusted user input to be sanitized and validated. Keep an open mind about what constitutes "input" in the context of pipelines.

For further reading on the perils of pipeline inputs, take a look at Bishop Fox's work in [Poisoned Pipeline Execution Attacks: A Look at CI-CD Environments](https://bishopfox.com/blog/poisoned-pipeline-attack-execution-a-look-at-ci-cd-environments) and then go re-read some of your pipelines with a new perspective.

### Security contact management

The second control that deserves our attention is [OSPS-VM-02](https://baseline.openssf.org/versions/2025-02-25#osps-vm-02---the-project-must-publish-contacts-and-process-for-reporting-vulnerabilities) **_- The project MUST publish contacts and process for reporting vulnerabilities_**

I've worked in organizations of all sizes and whether they had 50-100 or 1000s of repositories, I've yet to see a coherent and well executed implementation that can quickly and easily answer the question - Who should be contacted as the responsible security owner of the code, configuration and dependencies in this Github repository?

When SCA and SAST tools scanning your repositories are reporting issues, you want to be able to answer that question quickly and with high quality. You probably also want the answer in a format that can be easily consumed by your automation tools.

While I have not implemented this pattern, I've wanted to setup [custom properties for repositories](https://docs.github.com/en/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization) in a Github organization and store a `security-contact` property on every repository.

How are you solving this problem today? If you don't have this problem, why not?

### Basic supply chain security hygiene

There are many controls in the baseline that address dependency-related risks in the software supply chain. I want to focus in on [OSPS-VM-05.03](https://baseline.openssf.org/versions/2025-02-25#osps-vm-0503) **_- While active, all changes to the project's codebase MUST be automatically evaluated against a documented policy for malicious dependencies and known vulnerabilities in dependencies, then blocked in the event of violations, except when declared and suppressed as non-exploitable_**

A common pattern to meet this requirement is for CI pipelines evaluating pull requests to identify changes to `package.json` or `requirements.txt` and enforce additional manual or automated security review of the dependency changes against policy. Take a look at the OpenSSF project [Minder](https://mindersec.github.io) for one potential solution to the dependency policy evaluation and enforcement automation. Another potential solution approach if you are dealing with Node and Python projects is to consider using [Supply Chain Firewall](https://github.com/DataDog/supply-chain-firewall) within your CI pipeline environment and gain both additional observability as well as automated policy-based control.

## Conclusions

The baseline controls are provocative (in a positive way) in their scope if we consider all 3 levels of controls and the breadth of categories they span: Access control, Build and release, Documentation, Governance, Legal, Quality, Security assessment and Vulnerability management. Many of the companies who originally supported the creation of OpenSSF are likely not rigorously adhering to each and every one of these controls in the open source projects they contribute to. Meeting the requirements of each control takes prioritization and investment, but the goal of the baseline and its usage isn't to meet 100% of the controls, it is to assess where we are today in our pursuit of risk reduction and what we could do today or tomorrow to reduce risk further.

I choose to react positively to that provocation. As consumers of and contributors to open source we should both look to evaluate the security of those projects and understand the concrete steps we can take to improve the security of those projects. Ideally, we then pivot those learnings into contributions both to the community, but also in the commercial, closed source settings we likely work in, driving meaningful risk reduction as a result.
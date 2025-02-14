<!-- omit in toc -->
# DISW CICD Blueprint

The goal is to define standardized steps to be used across all DISW pipelines. In the near term, this should be used as a guide to developing pipelines for the selected applications. In the long term this can be adapted into customer-facing documentation on what good pipelines look like.

<!-- omit in toc -->
## CICD Dashboard/Console

There is a missing piece to our overall vision, which is a dashboard of CICD health/state across multiple products. This will be a complex thing, and not in our initial push, but we need to keep this in mind as we develop these jobs.

We must take special consideration into "validation" of each stage. How can we validate that teams are doing the required steps? Some of them might not have a good answer, but we should make an effort have validation for jobs where it makes sense. Eventually the CICD dashboard would consume this validation information.

<!-- omit in toc -->
## Project & General Guidelines

<!-- omit in toc -->
### Project Configuration

All projects must:

- Be configured with protected branches and (where applicable) tags.
- Enable approvals on any merge to a protected branch.

Projects should:

- Leverage code-owners for approvals
- Use Code Sentinel (WIP) for automated security and code review
  - This is an internal tool, which is not available for general usage yet

GitLab has introduced [CIS Benchmarks for GitLab projects](https://about.gitlab.com/blog/2024/10/29/new-cis-gitlab-benchmark-scanner-boosts-security-and-compliance/). This should be used to assess project configuration & health.

<!-- omit in toc -->
### General

- Pipelines should be run in clean environments (not re-used)
  - Leverage Docker containers where possible
  - Leverage CDE VM provisioning logic where VMs are required.
- All projects should leverage Renovate.
- Some of the below steps may not be applicable, but we need to take a critical look and make sure we're ready to explain and defend that.
- All provisioned environments should leverage Infrastructure As Code (IAC). There are _many_ tools available that can assist, depending on the environment. Examples: Terraform, Ansible, Puppet, ArgoCD, Helm, etc.
  - All IAC code should be treated as production code with regards to testing, validation, security checks/scans, etc.
  - As our pipelines mature, we will integrate policy checking and enforcement at the IAC layer, leveraging tools like Open Policy Agent (OPA).

<!-- omit in toc -->
## The Plan

We have identified two projects with which we want to build out a pipeline to this blueprint. We will identify two people from each team as the "facilitators" to this effort. These individuals do not have to complete this task by themselves. They have the authority to pull in any other team members to assist with steps.

The two applications are:

- SBOM Juicer
- CI4DMS (Pat Bevak's team)

<!-- omit in toc -->
### Steps & Deliverables

1. Analyze existing pipeline and create plan of execution to move to this pipeline
   1. **Deliverable** Documented plan of attack
2. Implement pipeline improvements
   1. **Deliverable** Document of pipeline steps, their purpose, any challenges/shortcomings (especially when compared w/ this blueprint)

As we go through this exercise, it would be good to feed back into the blueprint. Where can we add more details? What validation steps/data could we add, or need?

<!-- omit in toc -->
## Pipeline Jobs

These jobs are intentionally left outside of a build stage. The stages are customizable, and we should  leverage parallelization where it's possible. If the below jobs work well combined, we can combine as well. This is meant to be a list of required steps and their details, not necessarily a map of how to organize those steps.

We should also consider any DevX improvements at every opportunity. Leverage existing MR widgets, MR Decorator, Rennovate, etc.

We should consider when each stage runs:

- Every commit - Needs to be fast & easy
- Only on main branches - Can be more secure, longer-running.
- Only on tags/releases/etc. - Most secure, most reliable, etc.

For consistency, these pipelines will use the following envrionment designations for any steps related to deployed/runtime environments:

- Development - multiple environments (review apps) stood up as part of a feature branch pipeline
  - Used for integration testing, code review, etc.
- Pre-production - single environment based on latest changes of "main" branch
  - In the event where a team needs multiple "preprod" environments, they should name the environments with their purpose. "Integration-test" as an example.
- Production - single environment based on latest "tag"

 As we onboard customers, there may be more environments they require. We'll assess that as it comes up.

- [Template](#template)
- [Secret Scanning](#secret-scanning)
- [Lint](#lint)
- [Unit-Test](#unit-test)
- [SAST](#sast)
- [DAST](#dast)
- [Build](#build)
- [Docker Build](#docker-build)
- [Docker Vuln. Scan](#docker-vuln-scan)
- [SBOM Generation](#sbom-generation)
- [API Testing](#api-testing)
- [API Registration](#api-registration)
- [API Documentation Publishing](#api-documentation-publishing)
- [Product Registry](#product-registry)
- [Plagiarism Checks](#plagiarism-checks)
- [SBOM Juicer (SCA)](#sbom-juicer-sca)
- [Version](#version)
- [Deploy](#deploy)
- [Rollback](#rollback)
- [Integration Testing](#integration-testing)
- [Acceptance Testing](#acceptance-testing)
- [CIS Benchmarks](#cis-benchmarks)
- [Arch. Tech Debt Eval](#arch-tech-debt-eval)
- [OSS Clearance Validation](#oss-clearance-validation)
- [Fuzzing](#fuzzing)
- [Pen Testing](#pen-testing)

### Template

Each of the following job definitions will follow this template.

- **Trigger or Build Env.** - Every Commit, Specific Env., etc.
- **Allow Bypass or Failure?** - No, Yes, Special case.
- **Standard or Custom?** - Is this a standard step we can use off-the-shelf (common services...), or something to be customized.

Additional job description/details.

<!-- omit in toc -->
#### Validation & Monitoring

What data are we expecting out of the job that could be used to verify the job was successful, and requirements met?

<!-- omit in toc -->
#### Tools

- ToolA - Purpose (if unclear)
- ToolB

### Secret Scanning

- **Trigger or Build Env.** - Every Commit
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - OTS

Run secret scanning, leveraging the [existing template](https://gitlab.industrysoftware.automation.siemens.com/DevOps/cde/templates-examples/templates-library/secret-scanning).

[Examples of the secrets manifest](https://gitlab.industrysoftware.automation.siemens.com/DevOps/cde/templates-examples/microserviceframeworkdemo/cde-webapp-endpoint-java/-/blob/new_feature/.secret-scan-config.toml?ref_type=heads)

If possible - we should leverage the existing MR widgets to display findings

For feature branches, we can do a "diff" scan - only scan the affected files.

For main branches or tags, we should do a "full" scan - scan _everything_. This might not be feasible for large projects. Maybe we have smarter logic (only scan changes since last change), or move to a scheduled scan.

<!-- omit in toc -->
#### Validation & Monitoring

As part of the secret scanning project/template, each project has a "secrets manifest" which enables teams to ignore existing secrets in code.

**Eventually** this should be uploaded to a centralized system, where the security team can assess what is being ignored where & why.

For now - maybe this is saved as a pipeline artifact.

<!-- omit in toc -->
#### Tools

- GitLeaks
- CDE CI Template

### Lint

- **Trigger or Build Env.** - Every Commit
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - custom

This step should include Source, IAC, etc.

<!-- omit in toc -->
#### Validation & Monitoring

At a minimum, it would be good to know what linters are being run against the project. We don't have anywhere to send this data yet.

<!-- omit in toc -->
#### Tools

- Lanugage specific linters
- IAC linters
- Generic Readme/yaml linting
- mega-linter is useful for packaging many linters into one step, but it is also slow/cumbersome. Up to the implementer to choose to manually have different lint jobs or use mega.

### Unit-Test

- **Trigger or Build Env.** - Every Commit
- **Allow Bypass or Failure?** - This could probably allow failure, but potentially have some threshold?
- **Standard or Custom?** - custom

For larger projects, look into running unit tests against changed code rather than all code.

<!-- omit in toc -->
#### Validation & Monitoring

- Unit test KPIs (uploaded to SQ?):
  - Code coverage - should be uploaded to GitLab as a badge. [Badge Docs](https://docs.gitlab.com/ee/user/project/badges.html)
  - Pass/fail percentage

<!-- omit in toc -->
#### Tools

- Language specific
- Sonarqube for KPIs.

### SAST

- **Trigger or Build Env.** - Every Commit. Especially including releases, tags, etc.
- **Allow Bypass or Failure?** - Not sure about Coverity. SQ should fail if we break quality profiles.
- **Standard or Custom?** - custom

Must be Coverity or SQ.

SQ should leverage the MR widget/feedback mechanism.

[Related XCD Docs](https://developer.internal.siemens.com/fds/p0/xcd/codequalityanalysis/cqa-sonarqubeintegration.html)

<!-- omit in toc -->
#### Validation & Monitoring

Eventually I'd like to have a link to the SQ dashboard in the "CICD Console", which grabs KPIs against the project.

Renata's team has experience with wrapping SQ to create additional dashboards/views.

TODO: Investigate what could be used from Coverity.

<!-- omit in toc -->
#### Tools

- Sonarqube
- Coverity

### DAST

- **Trigger or Build Env.** - Up to the application. At least Preprod
- **Allow Bypass or Failure?** - TBD.
- **Standard or Custom?** - custom

DAST can be a destructive thing to an envrionment. It requires a high-functioning team/application to be able to spin up environments on-demand for this type of testing.

We have not identified a standard tool. Burp Suite has been used by other teams in Siemens for this type of testing, but it's not licensed outside of DM.

<!-- omit in toc -->
#### Validation & Monitoring

Test KPI:

- Number of tests
- Pass/Fail

<!-- omit in toc -->
#### Tools

- Burp Suite

### Build

- **Trigger or Build Env.** - Every Commit.
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - custom

Artifacts must be pulled from internal repository (Artifactory). Eventually this will be moved to the OSS repository, but that is not available currently.

TODO: There are pre-requisite steps to do w/ Artifactory to generate SBOMS. Need to insert those here.

<!-- omit in toc -->
#### Validation & Monitoring

KPIs:

- Build time
- Environment information

<!-- omit in toc -->
#### Tools

- Artifactory
- Docker

### Docker Build

- **Trigger or Build Env.** - Any change that impacts container contents should rebuild the container
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - Standard if possible, but will run into scenarios where it doesn't work.

At a minimum, containers should be built and pushed to GitLab. Optionally they can be pushed to Harbor if they will be consumed in XCR or similar.

<!-- omit in toc -->
#### Validation & Monitoring

TODO: Figure out what we want to do here. We want _something_ in the CICD dashboard, but unclear what.

<!-- omit in toc -->
#### Tools

- Docker
- GitLab
- Harbor
- [Container build pipeline (XCD)](https://gitlab.industrysoftware.automation.siemens.com/chimera/container-pipeline)
  - [Example](https://gitlab.industrysoftware.automation.siemens.com/DevOps/cde/mrdecorator/mrdecorator)

### Docker Vuln. Scan

- **Trigger or Build Env.** - Any pipeline that builds a container should scan for vulns.
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - OTS - might be covered by Container build pipeline.

<!-- omit in toc -->
#### Validation & Monitoring

- Vulnerability report available in pipeline artifacts, MR widget or registry UI.

<!-- omit in toc -->
#### Tools

- XCD Container build pipeline
- Trivy
- Aqua

### SBOM Generation

- **Trigger or Build Env.** - Any pre-prod or prod environment
  - Any "main" branch, or any tags
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - long term goal should be OTS, understand we might not have templates for every step yet.

We want to have SBOMs for anything/everything we can. Some potential options:

- From package manager/code (Artifactory)
- From CI/Build environment
  - Specifically interested in "build" stage
- From container
- From BDBA (if we're building binaries)

At this point, more data is better, and we'll figure out how what to do with it all later.

<!-- omit in toc -->
#### Validation & Monitoring

- SBOM should be saved somewhere _persistent_
  - Juicer
  - CI Artifacts ()
  - Repository
  - GitLab Release

<!-- omit in toc -->
#### Tools

- Artifactory
- BDBA
- Docker

### API Testing

- **Trigger or Build Env.** - Any change that impacts API code/logic
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - Long term goal is OTS.

<!-- omit in toc -->
#### Validation & Monitoring

- KPIs about
  - how many tests
  - pass/fail percentage
  - percentage of APIs tested

<!-- omit in toc -->
#### Tools

- SmartBear (we have a trial license we could get ahold of)
  - There is a pipeline that uses smartbear we could potentially steal.
- OK with manual/coded tests for now.

### API Registration

- **Trigger or Build Env.** - Production, Pre-prod
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - Long term goal is OTS.

This is currently only applicable for production public facing APIs deployed to XCR. In the future we'll have on-prem and pre-prod environments to do this.

<!-- omit in toc -->
#### Validation & Monitoring

TODO: Possibly query the API GW for registration endpoint from the CICD console.

<!-- omit in toc -->
#### Tools

- Kong
- AWS GW

**Note:** There is a good bit of discussion what the final offering will be for API GW.

### API Documentation Publishing

- **Trigger or Build Env.** - When production APIs are available.
- **Allow Bypass or Failure?** - No.
- **Standard or Custom?** - Long term goal is OTS.

For internal APIs - publishing Swagger with the application is probably sufficient.

**TODO** Figure out how/where we're publishing documentation today for customer-facing APIs.

<!-- omit in toc -->
#### Validation & Monitoring

- Verify documentation publishing successful

<!-- omit in toc -->
#### Tools

- TBD

### Product Registry

- **Trigger or Build Env.** - Production versions/tags
- **Allow Bypass or Failure?** - Manual job possible for complex teams/patterns
- **Standard or Custom?** - Standard template

JSON file stored on the project with all the metadata, use the snippet to upload it to the registry.

<!-- omit in toc -->
#### Validation & Monitoring

What data are we expecting out of the job that could be used to verify the job was successful, and requirements met?

<!-- omit in toc -->
#### Tools

- Product Registry CI snippets (TODO!)

### Plagiarism Checks

- **Trigger or Build Env.** - At a minimum preprod and prod environments. I do not know the impact of running this on every build.
- **Allow Bypass or Failure?** - TBD. I'm not sure what BDHub will return to the pipeline.
- **Standard or Custom?** - TBD.

We should use BDHub to do the plagiarism checks. Any other information we can gather (SCA, SBOM) should also be saved as an artifact.

<!-- omit in toc -->
#### Validation & Monitoring

As of now, the results are uploaded into BlackDuck hub, not available in the pipeline itself.

<!-- omit in toc -->
#### Tools

- BlackDuck Hub

### SBOM Juicer (SCA)

- **Trigger or Build Env.** - prod environments.
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Standard templates

<!-- omit in toc -->
#### Validation & Monitoring

Uploading to the juicer by itself is the validation.

<!-- omit in toc -->
#### Tools

- SBOM Juicer CI Snippets/Templates (TODO!)

### Version

- **Trigger or Build Env.** - prod environments.
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Standard template

[Version template](https://gitlab.industrysoftware.automation.siemens.com/DevOps/cde/templates-examples/templates-library/automatic-versioning)

<!-- omit in toc -->
#### Validation & Monitoring

N/A

<!-- omit in toc -->
#### Tools

- Auto-version CI snippets.

### Deploy

- **Trigger or Build Env.** - Dev, Preprod, Prod
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Custom

This step is potentially the most vague. We want to emphasize the importance of Automatically deployed and updated environments wherever possible.

This includes:

- Dev envrionments (feature branches)
- Pre-prod (main branch)
- Prod (Tagged Versions of the code)

This also includes ensuring the environment is provisioned, configured correctly, etc.

I understand that this depends heavily on how the application is deployed and in many cases the related systems. We should push to make this as complete as possible.

<!-- omit in toc -->
#### Validation & Monitoring

TBD - it would be good to have metrics for deployed envrionments in the CICD Dashboard, but not something critical at this point. The real validation of the environments happens in the integration-test job.

<!-- omit in toc -->
#### Tools

- Terraform
- Ansible
- GitLab Review-apps
- Artifactory
- Docker
- ArgoCD

### Rollback

- **Trigger or Build Env.** - Preprod, Prod
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Custom

We need to have the capability to rollback prod or pre-prod environments in the event that a deployment does not go well.

<!-- omit in toc -->
#### Validation & Monitoring

TBD - it would be good to have metrics for deployed envrionments in the CICD Dashboard, but not something critical at this point. The real validation of the environments happens in the integration-test job.

<!-- omit in toc -->
#### Tools

- Terraform
- Ansible
- GitLab Review-apps
- Artifactory
- Docker
- Argo

### Integration Testing

- **Trigger or Build Env.** - All deployed Env.
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Custom

This step is a bit open-ended.

We want to ensure that we're checking deployed environments to ensure they're healthy, performant, etc.

Long term, it would be good to have a suite of automated tests, especially for things like API based applications.

<!-- omit in toc -->
#### Validation & Monitoring

Test metrics should be available in CI artifacts.

<!-- omit in toc -->
#### Tools

- GitLab
- Any required tools for testing

### Acceptance Testing

- **Trigger or Build Env.** - Pre-prod (can be customized to fit application)
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Custom

Similarly to integration testing, this is a bit open ended.

What final checks can we run to ensure environment are ready for prod?

Should include:

- functional testing
- load testing

<!-- omit in toc -->
#### Validation & Monitoring

Test metrics should be available in CI artifacts.

<!-- omit in toc -->
#### Tools

- GitLab
- Any required tools for testing

### CIS Benchmarks

- **Trigger or Build Env.** - TBD
- **Allow Bypass or Failure?** - TBD
- **Standard or Custom?** - TBD

Evaluate any relevant environments/deployments against the CIS benchmarks. Depends heavily on the environment & application

<!-- omit in toc -->
#### Validation & Monitoring

TBD

<!-- omit in toc -->
#### Tools

TBD

### Arch. Tech Debt Eval

- **Trigger or Build Env.** - TBD
- **Allow Bypass or Failure?** - TBD
- **Standard or Custom?** - TBD

There are some tools that can give us some insight as to overall architectural technical debt. They might also provide information around Evaluate application against tools like [VFunction](https://vfunction.com/) or [Cast](https://www.castsoftware.com/imaging) for architectural technical debt.

<!-- omit in toc -->
#### Validation & Monitoring

TBD

<!-- omit in toc -->
#### Tools

TBD

### OSS Clearance Validation

- **Trigger or Build Env.** - Main branch, Releases
- **Allow Bypass or Failure?** - Probably need a "warn" in the event that things are not cleared
- **Standard or Custom?** - Standard is the goal

We're discussing this with the PSS team, so still very much a WIP, but we have decent plans for how to handle this with non-compiled code.

The plan is to have as much of the legal clearing process automated as possible. Where it's not possible to automate - we're hoping to leverage Arti and XRay to make it as easy as possible - pre-create documents/reports/etc. that a user can feed into the SCP process.

<!-- omit in toc -->
#### Validation & Monitoring

TBD

<!-- omit in toc -->
#### Tools

- Juicer
- Artifactory
- SCP

### Fuzzing

- **Trigger or Build Env.** - Dedicated test environment
- **Allow Bypass or Failure?** - No
- **Standard or Custom?** - Custom - at this point we don't have an identified fuzzing test suite/framework.

Fuzzing are tests that run "random" (typically coded/identified by testers) inputs against the application and see if it behaves as expected.

This typically modifies data within the application, so it's important to have the ability to efficiently provision and configure "test" environments.

<!-- omit in toc -->
#### Validation & Monitoring

TBD

<!-- omit in toc -->
#### Tools

- TBD

### Pen Testing

- **Trigger or Build Env.** - TBD
- **Allow Bypass or Failure?** - TBD
- **Standard or Custom?** - TBD

There has been an effort within Siemens to automate pen-testing in-house, to both save money and allow for more frequent testing.

There is a lot of unkowns in this space. We have Burp Suite as a potential tool that can be used, but it's a licensed tool.

More details will be filled in as they become available.

<!-- omit in toc -->
#### Validation & Monitoring

TBD

<!-- omit in toc -->
#### Tools

- TBD

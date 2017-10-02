--- 
layout: post
title: "DevOps - An Implementation Strategy"
author: "Author"
comments: true
tags : [DevOps, Automation, Strategy]
---

__Goal__

Implementing DevOps in Spider. Brainstorming Session.

#### Background 

We would like to see what we can do to get more 'DevOps'y.

-   Culture
-   Tools

We are currently looking at the tool options. Goal is to identify subset of __tools__ and __practises__ that we should start off with.

__Key Drivers - Context__
-   Spider is Azure oriented.
-   Spider deals with .NET, WEB and Mobile on Azure, primarily. - *big assumption here ??*
-   *more??*

__Key Drivers - Basic DevOps Practises__
-   One Step Test/Build/deploy AKA Continuous Delivery.
-   Automation.
-   Feedback Loops AKA Shared Dashboards - *these shared dashboard seem key to implementing devops and also the most problematic*

## SDLC Stages
This is from wikipedia, btw.

-   Code 
-   Test
-   Build/Package/Release  - *I have merged these three sections as the boundaries seem very difficult to draw*.
-   Monitor

There are several goals and working forces here 

## Tools / Practises / Activities by SDLC Stage

### Code

#### Practises
-   Version Control 
-   Development Workflows - viz. Gitflow - *possible activity here*
-   **Microservice architectures** - *possible activity here*

#### Tools 
- All your standard ones.

### Test 

#### Practises
-   Automation - Lots of it.
-   TDD.
-   BDD.

#### Tools
-   [Pex - Generates boundary conditions tests](https://docs.microsoft.com/en-us/visualstudio/test/intellitest-manual/introduction). - *(possible activity here - for viability check)*
-   Standard Unit Testing Frameworks viz. **NUnit**, MSTest, NSubstitute
-   Standard Integrated Testing frameworks - SpecFlow, Jasmine, Selenium, Mocha etc - *(possible activity to identify standard tooling)*
-   Metrics tools - NCover, Ndepend, Ncrunch - *(definite activity to estalish standard tools, metrics and reports)*
-   **Azure Dev/Test Labs**
-   Other tools - http://www.dotnetcurry.com/patterns-practices/1358/code-quality-tools

**Activities**
-   There seems to be no standards in the world around 
    -   Logging formats
    -   Test reports
-   Metrics which make sense
-   Metrics based on personas - *who needs what metrics?*
-   Automation ROI 
-   Azure Dev/Test Labs workshop.

**Gaps**
-   Product basic quality needs to be certified by Automated tests, not manual. 
-   Product quality metrics - coverage, boundary conditions, CC
-   specs/traceability. 
-   Standardized test reports - Ideally automated.
-   Local Automated Tests for developer feedback.

### Build/Release/Package (BRP in short)

#### Practises
-   One Touch Build and Deploys.
-   Build Pipelines - *(possible activity here)*
-   Standardized resources for builds - *(possible activity - samples and start-up kits etc)* 
-   Same builds on dev and test machines. 
-   Setup builds/Automated local deployment.
-   Complex Testing Needs.
-   Advanced Process Automation.
    -   Automated provisioning - particularly useful in cloud scenarios.
    -   Infrastructure configuration and management
    -   Infrastructure as Code tools

#### Tools

-   MSBuild
-   Powershell,
-   PSake,
-   MSDeploy,
-   DACPAC,
-   WIX,
-   VSTS.
-   Puppet/Chef - Desired State Configuration tools
-   Ansible/OctopusDeploy - Multi Step Deploy tools.
-   ARM Templates,
-   Docker
-   *(others??)*

#### Activities
- workshops and trainings on the following
    -   PowerShell
    -   ARM templates
    -   VSTS?
    -   Docker
    -   *Puppet/chef/ansible ??* - Am not sure if we should even go after these.

## Monitoring

BIG PROBLEM AREA HERE - This is the one we are probably going to have the most difficulty with.

### Practises 
-   Feedback Loops.
-   Shared Dashboards.
-   Operation Trends

### Tools
-   Azure AppInsight,
-   NewRelic,
-   ELK,
-   Azure OMS
-   Google Analytics

### Activities
-   Workshops on the tools. *Almost all are new and untried*
-   Activity to Establish 
    -   Views for Dashboards.
    -   Personas for Dashboards
    -   Metrics by View/Persona
    -   Data Collection and Aggregation

## All Activity list so far

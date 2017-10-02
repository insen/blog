--- 
layout: post
title: "DevOps - Making a Start !"
author: "Author"
comments: true
tags : [DevOps, Automation, Strategy]
---

DevOps is everywhere these days. As it should be, yes. Yet that doesn't take anything away from the fact that DevOps is a difficult concept to understand and implement. The principles are fair and easy enough to grasp, but then the *where to begin* question comes in, and that is a very difficult question for something as nebulous as DevOps.

So how does one start being 'DevOps'y?  What drives DevOps?

## Context, Culture, Tools, Change, Automation, Feedback Loops and Collaboration ! 
The above set summarizes the key drivers of DevOps anywhere. Additionally, there are a small but definite set of foundational DevOps practises, which have been proven enablers of DevOps over time.

## Foundational DevOps Practises
-   One touch build - *there should be a single command to build and package the tested version of a system.*
-   One touch deploy - *there should be a single command to deploy a tried and tested version to an environment.*
-   Automation - *everything repetitive should be automated.*
-   Monitoring Dashboards - *all process information, tests, builds, deploys, operations, features, analytics should be monitored and available.* 

Essentially, the practises above together cumulate into a CI/CD pipeline. How sophisticated/refined your CI/CD pipeline is, is one of the key technological measures of a DevOps oriented teams' maturity level.

So then the question driving a group trying to adopt a DevOps-y approach to software service delivery is, most likely, this

## Our DevOps Question
*Given our current context, what culture, practises and tools should I adopt which will enable*
*   *__faster change adoption__?*
*   *__more automation__?*
*   *__more shared process information__?*
*   *__more closed feedback loops__?*

*Note : Remember this question, We'll come back to this time and again.*

If we formulate an answer to the question above, we should have a workable Devops Strategy 101. Let us try an exercise and see. 

## Setting the '*Context*'
__Context__ is all about *NOT* sacrificing global optima to achieve local optima. *But then my junior dev comes up, the one I hired last month, saying he doesn't know what the ceo knows so he doesn't have all the context so he can't code a-la DevOps.* 

Oops! 

That didn't sound legit, did it? Cos it wasn't. In DevOps context itself is context-sensitive, i.e. it means take the biggest picture from where you stand. That's all you can do anyways. 

Since context is king, we will set a context. Assume the following organizational scenario within which to start out on a DevOps analysis first. 

*A team trying to provide custom software development and integration services on Azure. A team trying to a create a distributed processing layer for the .NET Stack system. A team working on big-data analytics and data presentation system on Azure. A management team trying to manage all these.*

*Also assume that all of the teams have various degrees of maturity on the build, test, deploy aspects of their individual systems - so effectively, automated builds, automated tests, and CI/CD exists, but to various degrees. Which is usually the case in diverse organizations.*

So the context here is a bunch of *software engineers trying to provide system development services on Azure*. 

Next.

## Consider 'Culture'
Now what can we do about Culture here?

Well, in a software development group, there is one *'Culture'* culture, or group culture or organization culture.

Now this top-level culture is probably not something everybody can start off with. But in every culture, there are sub-cultures. And there are sub-cultures across development tools, eco-systems or resources. *(Java vs .Net, anyone?)*. So for a start, let's just pick a culture by eco-system. Since Azure is common across, let's pick Azure. 

Enter *__Azure Culture__*. But what does *Azure Culture* in a DevOps organization mean? Let's go back and see how our original question changes.

*What practises and tools should I adopt which will enable*
*   *__faster change adoption__ in Azure?*
*   *__more automation__ in Azure?*
*   *__more shared process__ information in Azure?*
*   *__more closed feedback__ loops in Azure?* 

*Note - Should we limit ourselves to Azure specific tools. What if we work on Scala, Node, .NET? So perhaps, we need to analyse tooling more comprehensively.*

*The above is the kind of question that completely derails initiatives if we sit down to exhaustively analyse the options. Sometime you just pick the first option and go - just so that it sets an operational context, if nothing else.*

We will just set one - the .NET stack. *Keeping things within the family, you see. You pick scala, or hadoop or node as per your needs. After all, you need multiple iterations of this.*

So our task has now decomposed into a search for tooling options on the Azure platform and the .NET software development stack that enables foundational DevOps practises while achieving one or more of our goals - *faster change adoption, more automation,more shared process information, more closed feedback loops*. Once we identify tools and technologies, we *assess our maturity levels on identified tools and practises* to *identify __gaps__ which can be plugged*.

__The gaps are what we attack in our DevOps strategy 101__. The number of iterations of this process you go through, and the reviews with all concerned stakeholders should pare the list to items with highest priority overall.

Now we will do a non technical map-reduce. In *map* phase, we list out every possible tool, practise or activity that looks like it might help. In *reduce* phase - we prioritize items from list. 

And in map phase our goals are simple.
-   *__tools we can adopt__*
-   *__practises we can encourage__* 
-   *__activities we can do__*  

Now, the initial list will probably be large, as both Azure nor Microsoft .NET are huge eco-systems, but remember two things, 
-   We are just looking for what to start with.
-   As of now, we are just identifying as many options as we can that are applicable. 

But we do need to manage, classify and process this list. Wikipedia, in its DevOps page, suggests the SDLC stages - Code, Test, Build/Package/Relase (*merged these as boundaries between them are overlapping, especially in the matter of tooling*), and Monitor. 

### Azure/.NET Stack | SDLC Stage - Code
A basic DevOps practises here is *Version Control*. Distributed VCS are now standard, so we pick *__Git__*.

Architectural support targeting DevOps enablement for Azure PaaS systems is a much-needed eco-system centric practise. Enter *__Microservice architectures__* which by encouraging by encouraging small pieces and plug-and-play composition help keeping pieces small and nimble.

*__Test-Driven Design (TDD)__* and *__Domain Driven Design (DDD)__* are other standard practises that help in better code, decoupled pieces. 

#### Tools
-   Git.

#### Practises
-   Test Driven Design (TDD)
-   Domain Driven Design (DDD)
-   Microservices architectural style

### Azure/.NET Stack | SDLC Stage - Test 
Test automation has several flavours
-   Unit Tests
-   Integration Tests
-   Performance tests.

Additionally, we can also automate the test generation process through Behavior Driven Design Tools like SpecFlow. And test execution automation is usually done through build pipelines and Continuous Integration (CI)/Continuous Delivery(CD).
#### Tools
-   [Pex - Generates boundary conditions tests](https://docs.microsoft.com/en-us/visualstudio/test/intellitest-manual/introduction)
-   Unit And Integrated Testing Frameworks 
-   Identify Metrics and tools to report Metrics - NCover, Ndepend, Ncrunch
-   **Azure Dev/Test Labs**

Some possible activities. The one around *logging formats* and *test reports* is especially interesting - 
#### Activities
-   *__There seems to be no standards in the world around Logging formats and Test reports__*.
-   Persona based metrics - *who needs what metrics?*
-   Automation ROI graphs - *how to demonstrate?*
-   How to enable specification to final product traceability?

### Azure .NET Stack | SDLC Stage - Build/Package/Release
In this section, practises are common across software development verticals and horizontals.

#### Practises
-   One Touch Build and Deploys.
-   Build Pipelines - Same builds on dev and test machines, Setup builds and Automated local deployment.
-   Automated provisioning *(Infrastructure configuration and management and Infrastructure as Code tools)*

#### Tools
These include 
-   Automation tools (in and out-system) - Powershell, MSDeploy,  Puppet, Chef, Docker, Ansible, OctopusDeploy.
-   Build tools and Servers (in and out-system) - MSBuild, PSake, DACPAC, TeamCity, Jenkins. 
-   Eco System specific tools - Azure Resource Management (ARM) Templates and Azure CLI.

### Azure/.NET Stack | SDLC Stage - Monitor
This is probably a big problem area. For now we will gloss over it, but in practise, we would now probably take just monitoring as a problem area for DevOps and resort to the same technique we used throughout this post to break that down into tools, practises and activities. 

#### Practises
What is needed here is feedback loops and operation trend monitoring through shared dashboards. To large extents, these feedback loops across localized scope (from a DevOps point of view) can be provided by ALM tools like JIRA on the scope and requirements and features perspective, while build and test feedback loops can be provided by CI servers and build-pipelines. 

The difficult part is actually setting up on what to record and monitor. Sifting noise from signal here is a significant and *not-always-technical* step. And once past this, we have the technically demanding part in setting up a system which integrates separate local scopes into integrated dashboards. That is hard and definitely not DevOps 101. Maybe later.

Tools include VSTS - for application life-cycle management, Azure AppInsight for application operation management, standard external tools like NewRelic, and ELK, advanced services like Azure OMS (native) and Google Analytics (external).

#### Activities
Following activities are required.
-   Run Maturity Model evaluation on standard tools.
-   Establish Personas for Dashboards
-   Estrablish Metrics by View/Persona

At some point, build a __Data Collection and Aggregation Tooling/Implementation__.


## __*Big List of Tools, Practises and Activities*__
What we have been doing so far, is basically running the *map* part of a map-reduce analysis. Aggregating all of the tools, practises and activities found above, we get the following big list. 

### Tools / Practises
-   Git
-   Selenium
-   Build pipelines and CI
-   PowerShell 
-   Azure CLI 
-   Azure Dev/Test Labs
-   Azure Operations Management Services
-   Azure AppInsights.
-   Azure Application Resource Templates
-   VSTS
-   Google Analytics
-   NewRelic
-   ELK

# Practises
-   Microservices
-   TDD
-   DDD
-   BDD
-   Automated Unit Tests
-   Automated Integration Tests
-   One Touch Build and Deploys.
-   Build Pipelines - Same builds on dev and test machines, Setup builds and Automated local deployment.

### Activities
-   *__There seems to be no standards in the world around Logging formats and Test reports__*.
-   Persona based metrics - *who needs what metrics?*
-   Automation ROI graphs - *how to demonstrate?*
-   How to enable specification to final product traceability?
-   Maturity Model evaluation on identified tools.
-   Establish Personas for Dashboards
-   Establish Metrics by View/Persona
-   Establish Data Collection and Aggregation Tooling/Implementation.

This may not be exactly the list you come up with, but if you followed along on the exercise model, you have your own list. 

I am leaving the reduce part of this operation out, as that will probably diverge for everyone. But, at its basic premise, evaluating your group's maturity model on each of these items and identifying the highest priority items should get you there. Multiple rounds of reviews from multiple stake-holders is the way forward now.  But no of items should '*reduce*' in reduce phase.

So there you go, now we have a set of starting points for a DevOps Strategy 101. All we have to do now is Go forth, and Iterate.

Oh, and of course, implement !

------------------------------------------------------------------------------------------------------------
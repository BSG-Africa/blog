---
title: "Continuous Delivery And Pipelines"
author: vikrambindal
date: 2016-04-20 14:00:00 +0200
---

As software developers, we often strive towards greatness by adhering to design principles 
and patterns, applying enterprise frameworks that support the stack requirements of the 
application, following coding conventions and achieving high level of software quality through 
implementation of various testing paradigms present within the development community. This is 
great, but from a business point of view, we are often required to be agile in our approach, 
such that the software is ready for shipment at any given moment in time. At such a demand, 
various factors such as introducing new features, bug fixes, configuration changes, product 
enhancements, on-demand releases creep in that can affect the delivery process resulting the 
teams to question the reliability of quality, costs incurred and most importantly risks involved 
that can affect operations of a business.<!--more-->

#### Continuous Delivery 

The concerns raised above can be achieved using Continuous Delivery (CD). CD is an 
engineering approach which helps us develop software in such a way that it can be deployed 
to production at any time. The goal of CD is to keep software in a release state at any given 
time throughout its life cycle and enable a repetitive and reliable way to deploy software to 
any environment should the need arise. 

A typical strategy followed across teams is to use a build tool capable of performing common 
tasks such as build, test and custom tasks as a job on a daily nightly basis, and whenever a 
code is committed to a Source Code Management (SCM) repository. This enables the team to 
achieve software's health report, as well as taking preventive measure for failures on a pro-active 
basis. Some of the most common build tools being used are [Jenkins](https://jenkins.io/), [Go CD](https://www.go.cd), [Travis-CI](https://travis-ci.org/), [Circle-CI](https://circleci.com/) and many more such related products unique in their own offering with agent or agent-less architecture.

#### Pipelines

Varying between different projects, the build life cycle of a software can range from simple 
build-test-deploy pattern to rather complex ones in which case, monolithic flow of operations 
for a life-cycle isn't sufficient as one often requires tasks to be executed concurrently using 
the power of agent's spread across different machines performing task specific jobs. To create 
such complex workflows for a build life-cycle we use Pipelines. 

The pipeline, breaks down the delivery process into stages. Each stage is composed of one or 
more than one job that can execute concurrently. As the various stages are linked together, 
one often observes the process of "fan-out" and "fan-in" In a fan-out process, a job from a build 
stage for instance can be spawned to a next stage (assume testing stage as an example), in 
which multiple test jobs can be executed concurrently. In a fan-in process, various jobs 
executing concurrently when finished, can continue processing operations into the next stage 
of the flow in your delivery life-cycle.  A typical pipeline used forming a simple workflow 
is depicted in figure below:

<center>
  <img title="Typical Pipeline in CD" src="{{ site.baseurl }}/images/Pipeline.png"/>
</center>

From the figure above, the pipeline is composed of checkout, build, test and custom client 
specific tasks forming our build life-cycle. As can be seen there are multiple stages at play 
here. A common notion used in pipeline's are the keywords "up-stream" and "down-stream". 
Up-stream refers to the parent job of execution at a stage before the current stage where its 
materials/artefacts that maybe required down the life-cycle can be used. Down-stream refers to 
the stages composed of execution after its predecessor stage such that, it can consume the 
artefacts/materials from its parent relevant to continue its job. This is very much like a relay 
in athletics where a baton is passed from one runner to another, transitioning across multiple 
runners during the course of the race to reach the finish line. 

With release of Jenkins 2, the pipelines are bundled in as part of the distribution and offers 
ways to achieve these through DSL and scripting. Personally, I still prefer the various plug-ins 
offered through Jenkins 1 distribution that help create such pipelines. Some of the commonly 
used plug-ins are mentioned below and are often must-have to create a pipeline.

[Clone Workspace SCM Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Clone+Workspace+SCM+Plugin): 
This plugin helps in cloning the workspace of the parent/up-stream project into a downstream project.
[Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin): 
This plug-in is quite an essential and important plug-in to configure pipelines in Jenkins and 
offers various configurations and triggers on how the down-streams can execute from the 
up-stream tasks. 
[Delivery Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Delivery+Pipeline+Plugin): 
This plug-in serves as a view for us to visualize the pipeline build life cycle and see various 
elements as play such as time taken to complete each phase, job execution number, job status, 
job results reports (if any), etc. 
[Parameterized Trigger Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin): 
This plug-in offers the advantage of having a configurable build through the use of parameters a 
build may require to achieve flexibility in configuring the build.
[Join Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Join+Plugin): 
This plug-in helps in the fan-in process where concurrent tasks can collaborate before next series 
of tasks can execute. Through this, it ensures that while concurrent tasks for a certain phase are 
still executing, if one of them finishes then the propagation down the pipeline doesn't happen until 
all concurrent tasks in phases have finished executing. 
[Environment Inject Plugin](https://wiki.jenkins-ci.org/display/JENKINS/EnvInject+Plugin): 
This plug-in makes it easier to introduce user defined environment variables throughout the use of 
a build. This is especially useful when we read values from properties file and want to use that in 
our build process. 

The above listed plug-ins can be installed in Jenkins  through "Manage Plug-ins" sections under 
"Manage Jenkins" link if not already installed
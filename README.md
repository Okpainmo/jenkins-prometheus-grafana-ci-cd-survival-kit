# Jenkins-Prometheus-Grafana CI/CD Survival Kit.

> P.S: Life is already hard, tedious, and tasking enough as a startup CTO(who tirelessly leads work across the whole product development lifecycle - from ideation and design till deployments and DevOps), what's the point of stressing further. I did this - mainly to streamline and ease the repetitive task of creating multiple CI/CD pipelines on different projects. **I gave special attention into making this as convenient(and robust) as possible for myself. Feel free to bet your life on it.**

This repository is a survival kit for implementing CI/CD pipelines on different projects with Jenkins - Plus Prometheus and Grafana 
for Jenkins data visualization and monitoring. Starting with guides for implementations on Python and NodeJs project, I hope to proceed with 
implementations for other stacks that I'll work on e.g. The Rust/Axum stack.

The project is **meticulously** packed with all the commands, and PRACTICAL progressive steps required to fully implement robust CI/CD pipelines.

> Do well to drop a star, fork to own a copy, and contribute - to make the sheet more robust for everyone who will find it helpful.

## How You Can Contribute.

Just below are some ways you can contribute.

1. Add relevant screenshot(s) that describe each step that a command or process is implementing.

2. Update step explanations - to provide better comprehension for others.

3. Add new(**different**) Jenkins CI/CD workflows to make the project more robust and extra-helpful.

4. ...

## CI/CD Workflows.

This project contains multiple CI/CD workflow implementations, that can be basically classified into three(3).

1. A single virtual machine(VM) CI/CD workflow for small scale/startup projects.

    _Workflow Diagram._

2. A more robust CI/CD workflow with Docker as agent - for multiple VMs and/or server deployments.

    _Workflow Diagram._

2. An equally robust CI/CD workflow that uses Jenkins nodes as agent - this also for multiple VMs and/or server deployments.

    _Workflow Diagram._

> In general, after following and learning/practicing all the available workflows - when the project is 
> completed, you should become very vast with implementing robust CI/CD pipelines with Jenkins + Prometheus and Grafana(for monitoring/observability).
> By then, you should also, be very much able to think critically, combining the different possible-implementation options, to create the perfect 
> integration and deployment pipelines for your project.


**IMPORTANT.**

1. The cloud service provider used, is Amazon Web Services(AWS) - with EC2 for Virtual Machines(VMs)

2. The different workflows, will have implementations for one or more of the following:

    - Jenkins freestyle job.
    - Jenkins Pipeline Job - using the Jenkins pipeline GUI option(Pipeline script).
    - Jenkins Pipeline job - using a project Jenkins file(Pipeline script from SCM).

In terms of knowledge requirements, this project assumes that you.

1. Have considerable knowledge of some cloud and DevOps concepts and processes.

2. Already know how to deploy a server(NodeJs, Python, and more) on an AWS EC2 instance - using systemd for server management and persistence.

## Before You Contribute.

> This project will be a progressive one. I and any other contributor(s), will continue to add relevant updates and information. This makes it very important that you always share details about any contribution you wish to make - before-hand, and avoid the needless stress of proceeding to work a contribution for a topic that is already on the way or in-progress.

To contribute successfully, simply create a Github issues that mentions me, and I'll be right with you to discuss your proposed/intended contribution.

> Just in case this repository ever gets to save your butt at work or while learning Jenkins, and you wish to send an appreciation, [feel free to buy me a 'coffee'](https://paystack.com/pay/cagnddqmr2).

Cheers!!!

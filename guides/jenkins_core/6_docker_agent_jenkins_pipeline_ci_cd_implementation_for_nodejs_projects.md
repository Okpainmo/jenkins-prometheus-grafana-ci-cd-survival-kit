# Part 6: Implementing Docker Agent(Multi VM) *pipeline* CI/CD Workflows For NodeJs Projects.

In previous guides of the 'jenkins_core' section of this repository, I shared 2 single-VM CI/CD implementations - a *Freestyle* and a *Pipeline* implementation respectively. In this part, I'll be sharing the implementation of a **Jenkins Pipeline** CI/CD workflow for a NodeJs project - **using Docker as a Jenkins Agent**.

> A **Pipeline Jenkins Job** is a job/workflow defined using Groovy-based syntax, allowing for more complex and automated CI/CD workflows. It can be written in either the Declarative Pipeline syntax, which is simpler and more structured, or the Scripted Pipeline syntax, which offers greater flexibility and control. Pipeline jobs can be defined directly in Jenkins or in a Jenkinsfile stored in the project’s source code repository.

Since the CI/CD workflow is a an agentic pipeline build, this means we'd be needing at least 2 VMs. One to run the Jenkins Docker agent, and the other to run the main Jenkins controller.

Note the following:

1. For the Jenkins(Docker) agent: You should create an EC2 instance, and install Docker on it. This free [guide on EC2 Docker installation](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_docker-installation.md), is all you need.

2. For the Jenkins controller: You should create an EC2 instance, and make a clean/standard Jenkins installation on it. This free [EC2 Jenkins installation guide](https://github.com/Okpainmo/jenkins-prometheus-grafana-ci-cd-survival-kit/blob/main/guides/jenkins_core/1_installing_jenkins_on_aws_ec2.md), will be of immense help to you.

3. Also for the Jenkins(Docker) agent: Refer to this [guide on setting up Docker as Jenkins agent](https://github.com/Okpainmo/jenkins-prometheus-grafana-ci-cd-survival-kit/blob/main/guides/jenkins_core/5_setting_up_docker_as_jenkins_agent.md), and follow up with everything as expected. 

4. For both the Jenkins(Docker) agent, and the Jenkins controller, [Explore this guide](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/SSH_communications.md), and set up SSH communication between them and Github. 

> In progress
# Jenkins-Prometheus-Grafana CI/CD Survival Kit For Python And NodeJs Projects.

This project is a survival kit for implementing Ci/CD pipelines on Python and NodeJs project. It is **meticulously** packed with all the commands, and progressive steps required to fully implement robust CI/CD pipelines for the stated project stacks/types.

**IMPORTANT.**

This project assumes that you already have a good amount of cloud and DevOps knowledge.

It requires that you:

1. Have solid knowledge of some key cloud and DevOps implementations/practices.

2. Already know how to deploy a server(NodeJs or Python) on an AWS EC2 instance - using systemd for service management(and server persistence).

## Table Of Content.

- Part 1: Installing Jenkins On An AWS EC2 Instance.

- Part 2: Setting Up, And Connecting Jenkins With Github(Configuring Github Access).

- Part 3: Implementing Single VM CI/CD Workflows

  - Freestyle Implementation(For Both A NodeJs And Python Project Respectively).

  - Pipeline Implementation(For Both A NodeJs And Python Project Respectively).

- Part 4: Implementing Robust CI/CD Workflows With Docker As Agent.

  - Freestyle Implementation(For Both A NodeJs And Python Project Respectively).

  - Pipeline Implementation(For Both A NodeJs And Python Project Respectively).

- Part 5: Setting Up Prometheus And Grafana.

## Part 1: Installing Jenkins On An AWS EC2 Instance.

This step will guide through the process of installing a new Jenkins server on an AWS EC2 virtual machine.

**P.S:**

1. It is assumed that you already have an AWS EC2 instance created - which is also already running the Python/NodeJs project server. Both the project server, and the Jenkins server will run on the same EC2 instance.

1. Ensure to keep the VM port 8080 open. Jenkins runs by default on port 8080. Keeping everything behind a reverse proxy, will also be very beneficial

### 1.1. Update the system.

```bash
sudo apt update && sudo apt upgrade -y
```

_Screenshot needed._

### 1.2. Install Jenkins.

**Install Java Version 21(a Jenkins requirement) - Ensure to only install version 21 or higher. As at 24/04/2025, version 11 would not work with Jenkins. It has also been announced that Jenkins is already coming to an end of support for version 17.**

```bash
sudo apt install openjdk-21-jre -y # make the installation

java -version # verify the installation
```

_Screenshot needed._

### 1.3 Add Jenkins repo and install

 <!-- https://www.jenkins.io/doc/book/installing/linux/#debianubuntu -->

**Using the Jenkins 'Weekly Release' Installation**

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

_Screenshot needed._

### 1.4 Jenkins system chores - check Jenkins status, and restart if needed.

```bash
sudo systemctl status jenkins
# sudo systemctl start jenkins
sudo systemctl restart jenkins
# sudo systemctl stop jenkins
```

_Screenshot needed._

### 1.5 View the Jenkins logs - in case you need to troubleshoot.

```bash
sudo journalctl -u jenkins.service
```

_Screenshot needed._

### 1.6 Changing the Jenkins port.

**If Jenkins fails to start because a port is in use, run `systemctl edit jenkins` and add the following:**

```bash
[Service]
Environment="JENKINS_PORT=8081"
```

Here, "8081" was chosen but you can put another port available.

_Screenshot needed - show the above(port change) section as shown on this page - https://www.jenkins.io/doc/book/installing/linux/#debianubuntu_

### 1.6 Access Jenkins UI and perform necessary setups.

- Go to `http://<your-EC2-Public-IP>:8080`

_Screenshot needed._

- Get initial Jenkins password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

_Screenshot needed._

- Log-in.

- Finish setup wizard and install **suggested plugins**.

_Screenshot needed - show the option to select._

_Screenshot needed - show plugins installation in progress._

- Create **admin user**

_Screenshot needed._

## Part 2: Setting Up, And Connecting Jenkins With Github(Configuring Github Access).

From this point on, ensure to switch to the Jenkins user(on the EC2 VM).

```bash
sudo su - jenkins
```

_Screenshot needed._

### 2.1 Generate SSH key(on EC2).

Ensure to copy and save both the public and private key.

```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@ci" -f ~/.ssh/id_rsa # creates the keys
```

_Screenshot needed._

```bash
cat ~/.ssh/id_rsa.pub # reveals the public key
```

_Screenshot needed._

### 2.2 Add public key to GitHub repo

- Go to your GitHub repo → **Settings → Deploy keys**

> Ensure that the option to add 'Deploy Keys' is enabled/supported for your Github organization.

_Screenshot needed - show how to enable public keys on Github repos._

- Add the public key
- Check **Allow write access** if needed.

My recommendation: leave it unchecked - as an extra layer of safety for you repo.

_Screenshot needed - show how to add the public key - hence properly creating the new deploy key._

### 2.3 Test GitHub access on the EC2 VM(as `jenkins`)

```bash
sudo su - jenkins # you should have done this by now - as required at the beginning of part 2.
```

```bash
ssh -T git@github.com

# if the configuration is well implemented, this should return a response showing some details of the connected repo.
```

_Screenshot needed._

## Part 3: Implementing Single VM CI/CD Workflows

This section details the implementation of a single VM CI/CD workflow.

> By single VM, I simply mean, both the project server, and the Jenkins server all live on the same VM. This make the CD(deployment) process very straight-forward, and a lot easier. Perfect for small projects and startups.

_Workflow diagram needed._

> Similarly as required, the server(NodeJs or Python) should already be deployed on the AWS EC2 instance - using systemd for service management and persistence.

> For better understanding of the Ci/CD workflow, kindly study the workflow diagram above.

### 3.1 Freestyle Implementation(For Both A NodeJs And Python Project Respectively).

**NodeJs**

  1. Ensure all necessary plug-ins(Git and NodeJs) are Installed.

  _Screenshot needed._

  > While I'm not very sure, I believe that all the required Git plug-ins are installed at the start when you agreed to install **suggested plugins**. But still do well, to check for any Git and NodeJs plug-in you feel will be required, and install such.

  2. Create a New Freestyle Job.

  - Name: `your-project-job-name`

  - Type: select `Freestyle project`

  - Ensure to check Github project, then add the github repo URl - but in SSH format.

  _Screenshot needed - for the aspects covered so far._

  > IMPORTANT: While adding your Github URL during the jenkins job/project creation, ensure to use SSH format. E.g:
  >
  > **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. If you use the 'https' format(e.g: `https://github.com/your-github-org-name/repo-name`), your job will get stuck during https authentication step, as the Jenkins job, will request for authentication during the shell process.

  - Source Code Management.

    - Select **Git**

    - Repository URL: `git@github.com:your-username/your-repo.git` - **ensure to keep in SSH format**.

    - Credentials(for private or organizational repos): For private or org-based repos:

      - ensure that your Github org permits the use of personal access tokens for verifying third party processes, then create a new personal access token.

      - select username and password.

      - add your github username as the username

      - add the personal access token as the password.

    - Branch: `*/main` or `*/master` - or the branch you want the job to affect.

    _Screenshot needed - for the aspects covered so far._

  - Build triggers.

  For the job build triggers, there are 2 main options that we can conveniently use.

  I. Poll SCM: this creates(more like) a cron job, and polls to Github at whichever interval you set. If change is detected on the repo, the Jenkins job workflow it automatically triggered.

  **Setting your polling schedule**.

  - **`*/5 * * * *`**: this option polls every 5 minutes, but this may result in simultaneous polling across multiple jobs.

  - **`H/5 * * * *`**: Polls every 5 minutes but spreads the load evenly by assigning a random minute within the 5-minute window for each job, reducing the chance of overlapping polling times.

  This technique improves efficiency, especially when you have many Jenkins jobs with SCM polling, by minimizing the chance of simultaneous polling spikes and improving overall performance.

  _Screenshot needed - for the aspects covered so far._

  II. GitHub hook trigger for GITScm polling.

  This option(which is the one we'll be using), will require us to set up a webhook on Github. The webhook, will automatically send a message to Jenkins when ever a push is initiated to our repo. This will in-turn trigger Jenkins to automatically initiate the workflow process.

  **Configure Webhook in GitHub**

  - Go to your GitHub repo → **Settings → Webhooks**
  - Add new webhook:

    - Payload URL: `http://<EC2-Public-IP>:8080/github-webhook/
  E.g: `http://56.23.6.139:8080/github-webhook/`

    - Content type: `application/json`
    - Secret: leave blank
    - SSL verification: enable if your jenkins setup is configured on a standard domain/sub-domain, else disable.
    - Which events would you like to trigger this webhook?: select `Just the push event`.
    - Ensure to check `Active`.
    - Save/Add webhook.

# Jenkins-Prometheus-Grafana CI/CD Survival Kit For Python And NodeJs Projects.

This project is a survival kit for implementing CI/CD pipelines on Python and NodeJs project. It is **meticulously** packed with all the commands, and progressive steps required to fully implement robust CI/CD pipelines for the stated project stacks/types.

**IMPORTANT.**

This project assumes that you already have a good amount of cloud and DevOps knowledge.

It requires that you:

1. Have solid knowledge of some key cloud and DevOps implementations/practices.

2. Already know how to deploy a server(NodeJs or Python) on an AWS EC2 instance - using systemd for service management(and server persistence).

## Table Of Content.

- Part 1: Installing Jenkins On An AWS EC2 Instance.

- Part 2: Setting Up, And Connecting Jenkins With Github(Configuring Github Access).

- Part 3: Implementing Single VM CI/CD Workflows

  - Freestyle Implementation and Pipeline Implementation(For NodeJs Project).

  - Freestyle Implementation and Pipeline Implementation(For Python Project).

- Part 4: Implementing Robust CI/CD Workflows With Docker As Agent.

  - Freestyle Implementation and Pipeline Implementation(For NodeJs Project).

  - Freestyle Implementation and Pipeline Implementation(For Python Project).

- Part 5: Setting Up Prometheus And Grafana.

- Part 6: Optional Improvements(That You Can Contribute).

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

> For better understanding of the CI/CD workflow, kindly study the workflow diagram above.

### 3.1 Freestyle Implementation(For Both A NodeJs And Python Project Respectively).

**NodeJs(Freestyle Job)**

  1. Ensure all necessary plug-ins(Git, Docker, and NodeJs) are Installed.

  _Screenshot needed._

  > While I'm not very sure, I believe that all the required Git plug-ins are installed at the start when you agreed to install **suggested plugins**. But still do well, to check for any Git, Docker, and NodeJs plug-in you feel will be required, and install such.

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

      - If using an org repo, ensure that your Github org permits the use of personal access tokens for verifying external/third party processes.

      _Screenshot needed - show on Github - the area to allow the above stated access right._
      
      - create a new personal access token.

      - now back to jenkins, select username and password as authentication option.

      - add your github username as the username

      - add the new personal access token(the one you just created) as the password.

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

    _Screenshot needed - show(point to) webhook link._
    
    _Screenshot needed - show webhook creation area._

    **Configure Webhook in GitHub**

    - Go to your GitHub repo → **Settings → Webhooks**
    - Add new webhook:

      - Payload URL: `http://<EC2-Public-IP>:8080/github-webhook/
    E.g: `http://56.23.6.139:8080/github-webhook/`

      - Content type: `application/json`
      - Secret: leave blank - or add update as required.
      - SSL verification: enable if your jenkins setup is configured on a standard domain/sub-domain, else disable.
      - Which events would you like to trigger this webhook?: select `Just the push event`.
      - Ensure to check `Active`.
      - Save/Add webhook.

    **Select the branch to build**

    Set the appropriate branch 

    _Screenshot needed._

  - Environments.

    For the "Environment" option, ensure to select/check "Delete workspace before build starts". This will ensure that your Jenkins workspace is cleaned before any new workflow process kicks in. This saves from clogs and workspace cluttering.

    _Screenshot needed._

  - Build Step.

    Select "Execute shell", and add in the shell script below.

    > **P.S: the below shell script is quite detailed, so read-in to learn more about what is does. The workflow diagram for this section/workflow will also offer more insight. Also note that it is not perfect/complete script - as it leaves out very important chores like testing. It is however perfect and suited enough to perform the chore intended by this project - CI/CD. I hope to update and make it more robust subsequently. Kindly contribute if you can.**

    ```bash
      echo "[my-nodejs-freestyle-CI-CD-job] 🚀 Starting Jenkins job..."

      # access the Jenkins server workspace using $WORKSPACE
      cd "$WORKSPACE" || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ Failed to enter Jenkins workspace."
          exit 1
      }

      echo "[my-nodejs-freestyle-CI-CD-job] 🔄 Pulling latest changes from GitHub..."
      git pull origin main || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ Git pull failed."
          exit 1
      }

      echo "[my-nodejs-freestyle-CI-CD-job] 📁 Syncing the pulled/updated project content to live Node.js app folder - excluding unnecessary project files..."

      LIVE_APP_DIR="/home/ubuntu/your-nodes-project-directory"

      sudo rsync -av --delete \
        --exclude 'node_modules' \
        --exclude '.git' \
        --exclude '.env.local' \
        "$WORKSPACE/" "$LIVE_APP_DIR/" || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ rsync failed during code sync."
          exit 1
      }

      echo "[my-nodejs-freestyle-CI-CD-job] ✅ Code sync to $LIVE_APP_DIR completed."

      # ==============================================================================================
      # echo "[my-nodejs-freestyle-CI-CD-job] 🔐 Ensuring Jenkins has access to project directory..."

      # Add Jenkins to ubuntu group (only runs if not already a member)
      # sudo usermod -aG ubuntu jenkins

      # Fix ownership and permissions to allow group access
      # sudo chown -R ubuntu:ubuntu "$LIVE_APP_DIR"
      # sudo chmod -R 775 "$LIVE_APP_DIR"
      # ==============================================================================================

      # The above block is unnecessary - as we will give the Jenkins server all the necessary permissions directly on the host VM.

      # Load NVM(from the ubuntu user) for the Jenkins user to use Node.js installed via NVM - no need to install node directly on the Jenkins user. This process might be CPU intensive, so feel free to overwrite this step and install nodejs directly on the Jenkins server if you wish.
      export NVM_DIR="/home/ubuntu/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads NVM
      nvm use v22.12.0 || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ Failed to use Node.js version v22.12.0."
          exit 1
      }

      # Now proceed to the directory
      cd "$LIVE_APP_DIR" || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ Failed to enter project directory."
          exit 1
      }

      npm install || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ npm install failed."
          exit 1
      }

      echo "[my-nodejs-freestyle-CI-CD-job] 🔄 Restarting Node.js service..."
      SERVICE_NAME="your-nodejs-server-service.service"

      sudo systemctl restart "$SERVICE_NAME" || {
          echo "[my-nodejs-freestyle-CI-CD-job] ❌ Failed to restart $SERVICE_NAME"
          exit 1
      }

      echo "[my-nodejs-freestyle-CI-CD-job] 🩺 Checking service status..."
      sudo systemctl status "$SERVICE_NAME" --no-pager || {
          echo "[my-nodejs-freestyle-CI-CD-job] ⚠️ Service restarted but returned a non-zero status."
      }

      echo "[my-nodejs-freestyle-CI-CD-job] ✅ Deployment and service restart complete!"
    ```

    - Click 'Apply', then 'Save'.

    _Screenshot needed._

  3. Giving the Jenkins user the necessary permissions to run the necessary processes(i.e. the file syncing process, the server-restart, and the server-status-check) on the CI/CD workflow.

  Normally, the Jenkins user will not have direct access rights to run system services, so we'll need to update permissions on the host VM to permit the Jenkins to run the necessary system services.

    - Switch to the Ubuntu user and run the below command.

      ```bash
        sudo visudo -f /etc/sudoers.d/jenkins  
      ```
      the command will open nano CLI editor. Add the following to the opened file.

      ```bash
        jenkins ALL=(ALL) NOPASSWD: /usr/bin/rsync, /bin/systemctl restart your-nodejs-server-service.service, /bin/systemctl status your-nodejs-server-service.service
      ```

      **Breaking down the above script**

        | Part | Meaning |
        |------|--------|
        | `jenkins` | The Linux username (the Jenkins service typically runs as this user). |
        | `ALL=(ALL)` | This means the `jenkins` user can run the specified commands as **any user** on **any host** (standard sudo syntax). |
        | `NOPASSWD:` | Jenkins **won’t be asked for a password** when executing the commands listed after this. |
        | `/usr/bin/rsync` | Allows Jenkins to run the `rsync` command — typically used for syncing files from the Jenkins workspace to your Node.js app directory. |
        | `/bin/systemctl restart Zed-Labs-Platform-Server_PlatformCore01.service` | Allows Jenkins to restart your Node.js service (used for deploying updated code). |
        | `/bin/systemctl status Zed-Labs-Platform-Server_PlatformCore01.service` | Allows Jenkins to check the service status for health monitoring/logging. |

      **Nano Tips**
      
        - 1. press 'CTRL + o' - save the update
        - 2. press 'ENTER' - accept the file name prompt
        - 3. press 'CTRL + X' to exit Nano

  **In case the Jenkins user is not having access to the original nodeJs project directory, try the following**.

    a. **Add Jenkins to the `ubuntu` group:**

    ```bash
    sudo usermod -aG ubuntu jenkins
    ```

    b. **Set group permissions on the folder:**

    ```bash
    sudo chown -R ubuntu:ubuntu /home/ubuntu/your-nodes-project-directory
    sudo chmod -R 775 /home/ubuntu/your-nodes-project-directory
    ```

    c. **Reload Jenkins or re-login Jenkins session:**

    ```bash
    sudo systemctl restart jenkins
    ```

    This way:

    ✅ Jenkins can access the folder  
    ✅ Ubuntu retains ownership  
    ✅ More secure for shared environments

  4. Set Up Task Done: Now try building the CI/CD job.

**NodeJs(Pipeline Job - with Jenkins File)**

## 🔄 Optional Improvements

- Add credentials via Jenkins for secure deployment
- Archive artifacts or reports
- Add environment-specific steps (test/staging/prod)
- Integrate Slack/Discord for notifications



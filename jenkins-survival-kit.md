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


## Part 3: Implementing Single VM CI/CD Workflows

This section details the implementation of a single VM CI/CD workflow.

> By single VM, I simply mean, both the project server, and the Jenkins server all live on the same VM. This make the CD(deployment) process very straight-forward, and a lot easier. Perfect for small projects and startups.

_Workflow diagram needed._

> Similarly as required, the server(NodeJs or Python) should already be deployed on the AWS EC2 instance - using systemd for service management and persistence.

> For better understanding of the CI/CD workflow, kindly study the workflow diagram above.

> **P.S: Consider the below on Access the Jenkins workspace/filesystem**

```bash
# view the Jenkins file system local docker

cd /var/jenkins_home # access the volume holding the files

ls -ltr # list the files

cd workspace # access the Jenkins workspace, and see the files right from within docker.
```

**If in EC2 directly, the file system will be here**

```bash
cd /var/lib/jenkins
```

**If in on windows OS, the file system will be here(ensure to only use powershell, also ensure to run powershell as administrator - this is very important)**

```shell
cd C:\Users\Jenkins\AppData\Local\Jenkins\.jenkins\
```


### 3.1. Freestyle Implementation and Pipeline Implementation(For NodeJs Project).

#### 3.1.1 NodeJs(Freestyle Job)

##### 3.1.1.1 Ensure all necessary plug-ins(Git, Docker, and NodeJs) are Installed.

_Screenshot needed._

> While I'm not very sure, I believe that all the required Git plug-ins are installed at the start when you agreed to install **suggested plugins**. But still do well, to check for any Git, Docker, and NodeJs plug-in you feel will be required, and install such.

##### 3.1.1.2. Create a New Freestyle Job.

- Name: `your-project-job-name`

- Project Type: select `Freestyle project`

- Ensure to check Github project, then add the github repo URl - **but in SSH format**.

_Screenshot needed - for the aspects covered so far._

> IMPORTANT: While adding your Github URL during the jenkins job/project creation, ensure to use SSH format. E.g:
>
> **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. If you use the 'https' format(e.g: `https://github.com/your-github-org-name/repo-name`), your job will get stuck during https authentication step, as the Jenkins job, will request for authentication during the shell process.

- Source Code Management(SCM).

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

  > **P.S: the below shell script is quite detailed, so read-in to learn more about what is does. The workflow diagram for this section/workflow will also offer more insight. Also note that it is not perfect/complete script - as it leaves out very important chores like testing, and others. It is however perfect and suited enough to perform the chores intended to be handled this project - CI/CD. I hope to update and make it more robust subsequently. Kindly contribute if you can.**

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

##### 3.1.1.3. Giving the Jenkins user the necessary permissions to run the necessary processes(i.e. the file syncing process, the server-restart, and the server-status-check) on the CI/CD workflow.

1. Giving the Jenkins user permission to run system level services to manage our nodejs application. 

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
| `/bin/systemctl restart your-nodejs-server-service.service` | Allows Jenkins to restart your Node.js service (used for deploying updated code). |
| `/bin/systemctl status your-nodejs-server-service.service` | Allows Jenkins to check the service status for health monitoring/logging. |

**Nano Tips**

  - 1. press 'CTRL + o' - save the update
  - 2. press 'ENTER' - accept the file name prompt
  - 3. press 'CTRL + x' to exit Nano

2. Giving the Jenkins user the permission(s) to access our original project folder and perform actions on it. E.g. `npm install`

a. **Add Jenkins to the `ubuntu` group:**

```bash
sudo usermod -aG ubuntu jenkins
```

b. **Set group permissions on the folder:**

```bash
sudo chown -R ubuntu:ubuntu /home/ubuntu/your-nodes-project-directory
sudo chmod -R 775 /home/ubuntu/your-nodes-project-directory
```

```bash
sudo chown -R ubuntu:ubuntu /home/ubuntu/zed-labs-platform-server
sudo chmod -R 775 /home/ubuntu/zed-labs-platform-server
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

#### 3.1.2 NodeJs(Pipeline Job - with Jenkins File).

##### 3.1.2.1 Ensure all necessary plug-ins(Git, Docker, and NodeJs) are Installed.

Same as section 3.1.1.1.

##### 3.1.1.2. Create a New Freestyle Job.

- Create a temporary process directory for Jenkins.

When using Jenkins pipeline deployments, the shell processes differ from when it's a direct freestyle shell.

In short, Jenkins will attempt to create a duplicate of your project folder - but with an "@tmp" suffix.

Hence. if your project directory is "my-node-server", on the same root as your server, create another directory called "my-node-server@tmp", then grant Jenkins the necessary access to perform it's chores.

1. Switch to the ubuntu user if you're not there already.

```bash
sudo -su ubuntu
```
2. Create the @tmp directory.

```bash
mkdir my-node-server@tmp
```
3. Give necessary permissions.

```bash
sudo chown -R ubuntu:ubuntu /home/ubuntu/my-node-server@tmp
sudo chmod -R 775 /home/ubuntu/my-node-server@tmp
```

- Name: `your-project-job-name`

- Project Type: select `Freestyle project`

- Ensure to check Github project, then add the github repo URl - **but in SSH format**.

_Screenshot needed - for the aspects covered so far._

> IMPORTANT: While adding your Github URL during the jenkins job/project creation, ensure to use SSH format. E.g:
>
> **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. If you use the 'https' format(e.g: `https://github.com/your-github-org-name/repo-name`), your job will get stuck during https authentication step, as the Jenkins job, will request for authentication during the shell process.

- Pipeline.

  - Definition: Select "Pipeline script from SCM"

  > We'll use the other(direct) "Pipeline script" option for ONLY the single-VM docker agent Jenkins job of the NodeJs sectioN - for demonstration purpose. All other pipeline jobs will use the "Pipeline script from SCM" option.  

  > SCM => Source Code Management

    - SCM: Select Git

      > For everything in this section, refer and implement same as the Source Code Management(SCM) section of **section 3.1.1.2.**




## Part 6: Optional Improvements

- Add credentials via Jenkins for secure deployment
- Archive artifacts or reports
- Add environment-specific steps (test/staging/prod)
- Integrate Slack/Discord for notifications
- Add environment variables using **Credentials > Environment Injector Plugin**
- Use `.env` files securely with `dotenv`- VERIFY WHAT THIS MEAN - DOES IT RELATE TO JENKINS?.
- Add Slack notifications or GitHub status checks



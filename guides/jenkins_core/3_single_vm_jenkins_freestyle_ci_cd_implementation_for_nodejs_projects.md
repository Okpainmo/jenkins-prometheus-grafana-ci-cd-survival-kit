# Part 3: Implementing Single VM *freestyle* CI/CD Workflows For NodeJs Projects.

**P.S: If you know what you're doing, and already have things set up, then feel free to proceed. Else please stop here, and explore part 1(on installing Jenkins on an EC2 machine) and part 2(on configuring SSH communication between the Jenkins server and Github). You will find all of those in the 'jenkins_core' section of this repository**.

This section details the implementation of a single VM **Jenkins freestyle** CI/CD workflow for a NodeJs project.

> A **Freestyle Jenkins Job** is a job/workflow that is non-declarative. It is a job that works directly using shell scripting and not the declarative or declarative-file method that both uses the Groovy syntax.

> By single VM, I simply mean, both the NodeJs project server, and the Jenkins server all live on the same VM. This makes the CI/CD(especially the deployment) process very straight-forward, and a lot easier. Perfect for small projects and startups - even though I'll always recommend that each component stay on a separate VM. 
>
> Things will work great anyway. Just ensure that the EC2 VM is of a considerable spec, with good/high RAM and CPU. A free-tier EC2 VM is certainly not recommended for such a combination. Also, always put it in mind that with such a set up, distributed(node or agentic) jenkins builds, might become a necessity in a very short while.

## Workflow Diagram.

_Workflow diagram needed._

**The NodeJs project server should already be deployed on the AWS EC2 instance - using systemd for service management and persistence**.

> For better understanding of the CI/CD workflow, kindly study the workflow diagram above.

## Bonus Tips: Consider the below on accessing the Jenkins file-system/workspace.

1. **Docker**

```bash
# view the Jenkins file system local docker
cd /var/jenkins_home 

ls -ltr # list the files

cd workspace # access the Jenkins workspace, and see the files right from within docker.
```

2. **If on EC2 directly, the file system will be here**

```bash
cd /var/lib/jenkins
```

3. **If directly on a windows OS machine, the file system will be here(ensure to only use powershell, also ensure to run powershell as administrator - this is very important)**

```shell
cd C:\Users\Jenkins\AppData\Local\Jenkins\.jenkins\
```

## 3.1. Ensure all necessary plug-ins(Git and NodeJs) are Installed.

_Screenshot needed._

> While I'm not very sure, I believe that all the required Git plug-ins are installed at the start when you agreed to install **suggested plugins**. But still do well, to check for any Git and NodeJs plug-in you feel will be required, and install such.

## 3.2. Create a New Freestyle Job.

- Name: `your-project-job-name`

- Project Type: select `Freestyle project`

- Description:

  - Add a be-fitting text description for your project.

  - Ensure to check Github project, then add the github repo URl - but in SSH format.

  **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. 

  _Screenshot needed - for the aspects covered so far._

  > IMPORTANT: The Github verification/authentication will use SSH, hence it is important that you use the SSH remote repository path, for the github repo. I.e. when adding the repo link on the job creation process. Do not use the https url format.

  E.g:

  **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. 

  If you use the 'https' format(e.g: `https://github.com/your-github-org-name/repo-name`), your job will get stuck during https authentication step, as the Jenkins job, will request for SSH authentication during the shell process.

- Source Code Management(SCM).

  - Select **Git**

  - Repository URL: `git@github.com:your-username/your-repo.git` - **ensure to keep in SSH format as earlier stated**.

  - Credentials(for private or organizational repos): Easy and direct for public repos, unlike for private or org-based repos:

    - If using a private or an organization repo, ensure that your project or Github organization permits the use of personal access tokens for verifying external/third party processes.

    _Screenshot needed - show on Github - the area to allow the above stated access right._
    
    - create a new personal access token.

    - now back to jenkins, select username and password as authentication option.

    - add your github username as the username

    - add the new personal access token(the one you just created) as the password.

  - Branch: `*/main` or `*/master` - or the branch you want the job to affect.

  _Screenshot needed - for the aspects covered so far._

- Triggers.

  For the job build triggers, there are 2 main options that we can conveniently use.

  I. Poll SCM: this creates(more like) a cron job, and polls to Github at whichever interval you set. If a change is detected on the repo, the Jenkins job workflow will be automatically triggered and the CI/CD process will kick in. If no change is detected, no action will be taken.

  **Setting your polling schedule**.

  - **`*/5 * * * *`**: this option polls every 5 minutes, but this may result in simultaneous polling across multiple jobs.

  - **`H/5 * * * *`**: Polls every 5 minutes but spreads the load evenly by assigning a random minute within the 5-minute window for each job, reducing the chance of overlapping polling times.

  This technique improves efficiency, especially when you have many Jenkins jobs with SCM polling, by minimizing the chance of simultaneous polling spikes and improving overall performance.

  _Screenshot needed - for the aspects covered so far._

  > P.S: Study more above the polling schedule syntax system, and learn more about it's power and impressive flexibility.

  II. GitHub hook trigger for GITScm polling.

  This option(which is the one we'll be using), will require us to set up a webhook on Github. The webhook, will automatically send a message to Jenkins when ever a push is initiated to our repo. This will in-turn trigger Jenkins to automatically initiate the workflow process.

  _Screenshot needed - show(point to) webhook link._
  
  _Screenshot needed - show webhook creation area._

  **Configure Webhook in GitHub**

  - Go to your GitHub repo → **Settings → Webhooks**
  - Add new webhook:

    - Payload URL: `http://<EC2-Public-IP>:8080/github-webhook/
    E.g: `http://56.23.6.139:8080/github-webhook/`.

    The above assumes that you're using Jenkins directly without 
    any extra customization. Feel free to add a custom domain or sub-domain
    if you're set up one.

    E.g. `https://jenkins.mydomain.com/github-webhook`

    - Content type: `application/json`
    - Secret: leave blank - or add as required.
    - SSL verification: enable if your jenkins setup is configured on a standard domain/sub-domain, else disable.
    - Which events would you like to trigger this webhook?: select `Just the push event`.
    - Ensure to check `Active`.
    - Save/Add webhook.

  **Select the branch(es) to build**

  Type in the appropriate branch, and add any extra branch if necessary. 

  _Screenshot needed._

- Environments.

  For the "Environment" option, ensure to select/check "Delete workspace 
  before build starts". This will ensure that your Jenkins workspace is 
  cleaned before any new workflow process kicks in. This saves from clogs 
  and workspace cluttering.

  _Screenshot needed._

- Build Step.

  Select "Execute shell", and add in the shell script below.

  > **P.S: the below shell script is quite detailed, so read-in to learn more about what is does. The workflow diagram for this section/workflow will also offer more insight. Also note that this script is just an example/practice script. It is not perfect/complete script - as it leaves out very important chores like testing, and linting, app-building, and probably more. It is however perfect and suited enough to perform the chores intended to be handled this project - CI/CD. I hope to update and make it more robust subsequently. Kindly contribute if you can.**

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

    # Load NVM(from the ubuntu user) so that the Jenkins user can utilize the node installtion that I already made via NVM on the NodeJs project server. Feel free to use node directly if you are not using NVM like I did.
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

## 3.3. Giving the Jenkins user the needed permissions to run the necessary processes(i.e. the file syncing process, the server-restart, and the server-status-check) on the CI/CD workflow.

1. Giving the Jenkins user permission to run system level services to manage our nodejs application. 

Normally, the Jenkins user will not have direct access rights to run system services, so we'll need to update permissions on the host VM to permit the Jenkins to run the necessary system services.

- Switch to the Ubuntu user and run the below command.

```bash
  sudo su - ubuntu
  sudo visudo -f /etc/sudoers.d/jenkins  
```

The command will open nano CLI editor. Proceed to add the following to the opened file.

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

**Nano File-saving Tips**

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
✅ The Ubuntu group retains ownership  
✅ More secure for shared environments

4. Set Up Task Done: Now try building the CI/CD job.


# Part 4: Implementing Single VM *pipeline* CI/CD Workflows For NodeJs Projects.

**P.S: If you know what you're doing, and already have things set up, then feel free to proceed. Else please stop here, and explore part 1(on installing Jenkins on an EC2 machine) and part 2(on configuring SSH communication between the Jenkins server and Github). You will find all of those in the 'jenkins_core' section of this repository**.

While the previous guide(Part 3), featured a single VM *Freestyle* CI/CD implementation, this section details the implementation of a single VM **Jenkins Pipeline** CI/CD workflow for a NodeJs project.

> A **Pipeline Jenkins Job** is a job/workflow that is declarative. It is simply a job that works with the declarative Groovy syntax method - and if desired, a project Jenkinsfile.

> Similarly, By single VM, I simply mean, both the NodeJs project server, and the Jenkins server all live on the same VM. This makes the CI/CD(especially the deployment) process very straight-forward, and a lot easier. Perfect for small projects and startups - even though I'll always recommend that each component stay on a separate VM. 
>
> Just as stated in the previous part(Part 3) - things will work fine anyhow. Simply ensure that the EC2 VM is of considerable specs, with good/high RAM and CPU. A free-tier EC2 VM is certainly not recommended for such a combination. Also, always put it in mind that with such a set up, distributed(node or agentic) jenkins builds, might become a necessity in a very short while.

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

## 4.1. Ensure all necessary plug-ins(Git and NodeJs) are Installed.

> Same as for **Part 3.1** - refer to that part - inside the 'jenkins-core' folder of the repository.

## 4.2. Create a New Pipeline Job.

I. **Create a temporary process directory for Jenkins**.

When using Jenkins pipeline deployments, the shell processes differ from when it's a direct freestyle shell.

In short, for a Jenkins pipeline workflow, Jenkins will attempt to create a duplicate of your project folder - but with an "@tmp" suffix.

Hence. if your project directory is "my-node-server", on the same root as your server, create another directory called "my-node-server@tmp", then grant Jenkins the necessary access to perform it's chores.

1. Switch to the ubuntu user if you're not there already.

```bash
sudo su - ubuntu
```
2. Create the @tmp directory - basically an empty duplicate of your exact project folder on the same directory-level - but with an '@tmp' suffix.

```bash
mkdir my-node-server@tmp
```

3. Add the 'jenkins' to the `ubuntu` group.

```bash
sudo usermod -aG ubuntu jenkins
```

4. Give necessary permissions - read, write, and execute privileges to users in the ubuntu group(which now includes the jenkins user).


```bash
sudo chown -R ubuntu:ubuntu /home/ubuntu/my-node-server@tmp
sudo chmod -R 775 /home/ubuntu/my-node-server@tmp
```

II. **Name: `your-project-job-name`**

III. **Project Type: select `Pipeline`**

IV. Description:

  - Add a be-fitting text description for your project.

  - Ensure to check Github project, then add the github repo URl - but in SSH format.

  **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. 

  _Screenshot needed - for the aspects covered so far._

  > IMPORTANT: The Github verification/authentication will use SSH, hence it is important that you use the SSH remote repository path, for the github repo. I.e. when adding the repo link on the job creation process. Do not use the https url format.

  E.g:

  **`git@github.com:your-username/repo-name.git`** or **`git@github.com:your-github-org-name/repo-name.git`**. 

  If you use the 'https' format(e.g: `https://github.com/your-github-org-name/repo-name`), your job will get stuck during https authentication step, as the Jenkins job, will request for SSH authentication during the shell process.

V. **Triggers**.

  Similar to what was shared in Part 3, there are 2 main options we can conveniently use for the job build triggers.

  1. Poll SCM: this creates(more like) a cron job, and polls to Github at whichever interval you set. If a change is detected on the repo, the Jenkins job workflow will be automatically triggered and the CI/CD process will kick in. If no change is detected, no action will be taken.

  > **P.S: We can set this up directly in our Groovy script without doing it manually on the Jenkins UI. The Jenkins server will pick it up, and update on the UI - on our behalf. That will(at least) be a good test to verify that your Groovy script is being referenced.**

  ```groovy
    pipeline {
        agent any

        triggers {
            pollSCM('H/5 * * * *') // every 5 minutes
        }

        stages {
            stage('Example Step') {
                steps {
                    echo 'Polling example...'
                }
            }
        }
    }
  ```

  **Setting your polling schedule**.

  - **`*/5 * * * *`**: this option polls every 5 minutes, but this may result in simultaneous polling across multiple jobs.

  - **`H/5 * * * *`**: Polls every 5 minutes but spreads the load evenly by assigning a random minute within the 5-minute window for each job, reducing the chance of overlapping polling times.

  This technique improves efficiency, especially when you have many Jenkins jobs with SCM polling, by minimizing the chance of simultaneous polling spikes and improving overall performance.

  _Screenshot needed - for the aspects covered so far._

  > P.S: Study more above the polling schedule syntax system, and learn more about it's power and impressive flexibility.

  2. GitHub hook trigger for GITScm polling.

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

- **Pipeline**.

  - Definition: Select "Pipeline script"(for now).

  > We'll use the "Pipeline script" option and add our declarative Groovy script directly. But we'll only use that for testing our pipeline. Once everything is working as expected, we'll switch to the "Pipeline script from SCM" option and use a 'Jenkinsfile' that will be added to the root of our project repository on Github.  
  >
  > SCM => Source Code Management

    - SCM: Select Git

      > For everything in this section, refer and implement same as the Source Code Management(SCM) section of **Part 3.2.** inside the 'jenkins-core' folder of the repository

    - Script Path

      Simply add 'Jenkinsfile' - as we'll be using a Jenkins file on the root of our project.

    **See the Jenkinsfile Sample below**

    ```groovy
    pipeline {
        // agent { 
        //     docker {
        //         label 'docker-nodejs-agent'
        //     }
        // }
        agent any

        environment {
            TEMP_DIR = "/home/ubuntu/zed-labs-platform-server@tmp"
            LIVE_APP_DIR = "/home/ubuntu/zed-labs-platform-server"
            SERVICE_NAME = "Zed-Labs-Platform-Server_PlatformCore01.service"
            NVM_DIR = "/home/ubuntu/.nvm"
            NODE_VERSION = "v22.12.0"
        }

        stages {
            stage('Start') {
                steps {
                    echo "[zed-labs-freestyle-CI-CD-job] 🚀 Starting Jenkins job..."
                }
            }

            // stage('Clean Workspace') {
            //     steps {
            //         cleanWs()
            //         echo "[zed-labs] 🧹 Workspace cleaned."
            //     } 
            // }

            stage('Checkout Code') {
                steps {
                    dir("${env.WORKSPACE}") {
                        echo "[zed-labs-freestyle-CI-CD-job] 🔄 Pulling latest changes from GitHub..."
                        script {
                            def result = sh(script: 'git pull origin dev', returnStatus: true) // remember to specify the correct branch
                            if (result != 0) {
                                error("[zed-labs-freestyle-CI-CD-job] ❌ Git pull failed.")
                            }
                        }
                    }
                }
            }

            stage('Sync Code to Live Folder') {
                steps {
                    echo "[zed-labs-freestyle-CI-CD-job] 📁 Syncing the pulled/updated project content to live Node.js app folder - excluding unnecessary project files..."
                    script {
                        def result = sh(script: '''
                            sudo rsync -av --delete \
                              --exclude 'node_modules' \
                              --exclude '.git' \
                              "$WORKSPACE/" "$LIVE_APP_DIR/"
                        ''', returnStatus: true)
                        if (result != 0) {
                            error("[zed-labs-freestyle-CI-CD-job] ❌ rsync failed during code sync.")
                        }
                    }
                    echo "[zed-labs-freestyle-CI-CD-job] ✅ Code sync to $LIVE_APP_DIR completed."
                }
            }

            // stage('Load NVM and Use Node') {
            //     steps {
            //         script {
            //             sh '''
            //                 export NVM_DIR="$NVM_DIR"
            //                 [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"
            //                 nvm use $NODE_VERSION
            //             '''
            //         }
            //     }
            // }

            stage('Install Dependencies') {
                steps {
                    dir("${env.LIVE_APP_DIR}") {
                        script {
                            def result = sh(
                                script: '''
                                    export NVM_DIR="/home/ubuntu/.nvm"
                                    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" > /dev/null 2>&1
                                    nvm use v22.12.0 > /dev/null 2>&1  # Suppress nvm use output

                                    npm install > npm-install.log 2>&1
                                    exit $?
                                ''',
                                returnStatus: true
                            )

                            if (result != 0) {
                                echo "❌ npm install failed. Showing last 50 lines of log:"
                                sh 'tail -n 50 npm-install.log || echo "[log file missing]"'
                                error("[zed-labs-freestyle-CI-CD-job] npm install failed.")
                            } else {
                                echo "✅ Dependencies installed successfully. (output suppressed)"
                            }
                        }
                    }
                }
            }




            stage('Restart Service') {
                steps {
                    echo "[zed-labs-freestyle-CI-CD-job] 🔄 Restarting Node.js service..."
                    script {
                        def restartResult = sh(script: "sudo systemctl restart $SERVICE_NAME", returnStatus: true)
                        if (restartResult != 0) {
                            error("[zed-labs-freestyle-CI-CD-job] ❌ Failed to restart $SERVICE_NAME")
                        }

                        echo "[zed-labs-freestyle-CI-CD-job] 🩺 Checking service status..."
                        sh "sudo systemctl status $SERVICE_NAME --no-pager || true"
                    }
                }
            }
        }
        
        //  post {
        //     success {
        //         echo "[zed-labs-freestyle-CI-CD-job] ✅ Deployment and service restart complete!"

        //         dir("$LIVE_APP_DIR") {
        //             archiveArtifacts artifacts: "npm-install.log", onlyIfSuccessful: true
        //         }
        //     }
        //     failure {
        //         echo "[zed-labs-freestyle-CI-CD-job] ❌ Build failed. Check logs above for details."

        //         dir("$LIVE_APP_DIR") {
        //             archiveArtifacts artifacts: "npm-install.log", onlyIfSuccessful: false
        //         }
        //     }
        // }

        // or

        post {
            success {
                echo "[zed-labs-freestyle-CI-CD-job] ✅ Deployment and service restart complete!"
                sh "cp $LIVE_APP_DIR/npm-install.log $WORKSPACE/"
                archiveArtifacts artifacts: "npm-install.log", onlyIfSuccessful: true
            }
            failure {
                echo "[zed-labs-freestyle-CI-CD-job] ❌ Build failed. Check logs above for details."
                sh "cp $LIVE_APP_DIR/npm-install.log $WORKSPACE/ || true"
                archiveArtifacts artifacts: "npm-install.log", onlyIfSuccessful: false
            }
        }
    }
    ```
  




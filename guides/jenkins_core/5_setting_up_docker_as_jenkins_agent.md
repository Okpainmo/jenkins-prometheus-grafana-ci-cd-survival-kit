# Part 5: Setting Up Docker To Work as a Jenkins Agent.

You've got a grasp of Jenkins, and now you wish to set up Docker to work as a Jenkins cloud agent for your Jenkins controller?

This guide is a perfect tutorial to help you configure and set up a Docker agent setup that helps you handle Jenkins tasks/jobs.

For this guide, it is assumed that you have already created an EC2 VM with complete Jenkins installation - which will serve as the Jenkins controller. In case you haven't, this [EC2 Jenkins installation guide](https://github.com/Okpainmo/jenkins-prometheus-grafana-ci-cd-survival-kit/blob/main/guides/jenkins_core/1_installing_jenkins_on_aws_ec2.md) should help you out perfectly.

## Create An EC2 VM For The Jenkins Agent, and Install Docker.

Refer to my free github [guide on installing Docker on AWS EC2 machines](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_docker-installation.md).

## Docker Service Update And Jenkins Docker-Cloud Setup.

**The next couple of steps are very sensitive in nature, and as such, it is important that you only do this in a controlled environments. E.g. within a private network, or behind a proxy server like Nginx. A very viable option, is to restrict IP access(of any opened agent VM port) to only the IP of the VM running the Jenkins controller**.

1. **Edit Docker service file on EC2**:

```bash
sudo nano /lib/systemd/system/docker.service
```

Find the `ExecStart` line and change it from:

```bash
ExecStart=/usr/bin/dockerd -H fd:// # or whatever
```

To:

```bash
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

> The above, (kind of) hacks docker on the current(agent) VM, and grants access(via TCP) to a remote Jenkins controller, which can then connect and control the docker service/daemon REMOTELY - thus giving the Jenkins controller the ability to use it as an agent.

2. **Reload and restart Docker**:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
```

3. **Open port 2375 in your agent EC2 instance security group and grant access to ONLY the Jenkins master/controller VM IP address**. 

Example Use Case:

Allowing port 2375 access only from `198.51.100.10`:

* **Type:** Custom TCP
* **Port Range:** 2375
* **Source:** Change from `0.0.0.0/0` to `198.51.100.10/32` - Custom TCP(IPV4)

> Do well to add TLS(Transport Layer Security) and switch to the more secure port '2376' for docker - while still restricting access to only the Jenkins controller VM IP address.

4. On the Jenkins controller UI, go to **Jenkins → Manage Jenkins → Clouds → Add a cloud**

5. Select Docker(Options are AWS EC2, Kubernetes, Docker, and 'Copy Existing Cloud' - i.e. if you already created one) then give the cloud agent a name and proceed.

6. **On the "Docker Cloud details" drop-down/menu**

> In this part, you're basically connecting the exposed docker daemon/service of your proposed Jenkins client VM to the controller. That way, the remote Jenkins controller is able to delegate jobs/tasks to it - controlling it's docker daemon/service remotely.

    - Docker Host URI

        - Add 

        ```bash
        tcp://ip_of_jenkins_agent_vm:2375
        ```

    - Test the connection.

        - should return a response like below:

        ```bash
        Version = 28.1.1, API Version = 1.49
        ```

    - Tick 'Enabled', and save.

**On "Docker Agent templates" drop-down/menu**

> In this part, you're basically setting up how your Docker agent will work. You're specifying the type of image that would be run, and other key details. Docker cloud agents are powerful. The flexibility that they bring to Jenkins, are simply mind-blowing.

    - Add a label(identifier) for the docker agent template.
    - Tick 'Enabled'.
    - Add a name - usually same as label above.
    - Add the docker image to use on the agent(the beauty of Docker as a Jenkins agent - flexibility of environments, and parallelism). 

    E.g.

    ```bash
    okpainmo/jenkins_nodejs_agent_linux_deb:6.5.25
    ```

    - Set instance capacity - the number of containers the agent is allowed to spin(3 - 5 should be good I guess).
    - Remote File System Root.

    ```bash
    /var/jenkins_home 
    ```

    - Study(click the '?' marks on the Jenkins UI) other options and make appropriate choices - especially for "Idle timeout" and "Stop timeout".
    - Apply and Save.
    - Then use in your declarative pipeline scripts.

    ```groovy
    pipeline {
        agent {
            docker {
                label 'agent_name'// e.g. 'my_docker-agent'
            }
        }

        stages {    
            stage('Build') {
                steps {
                    sh 'whoami'                   
                }
            }
        }
    }
    ```

**Study on the differences between, and the pros/cons of using Docker as a Jenkins CLOUD agent, and using a Jenkins NODE(SSH) agent.**


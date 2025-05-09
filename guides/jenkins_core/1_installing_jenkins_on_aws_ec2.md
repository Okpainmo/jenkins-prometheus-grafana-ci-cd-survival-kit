# Part 1: Installing Jenkins On An AWS EC2 Instance.

This guide details out - in very thorough fashion, how to install Jenkins on an AWS EC2 virtual machine(VM).

**P.S: The installation to be performed - which is how I recommend installing Jenkins on a VM, is not with Docker. The installation is a direct installation on the Linux machine. Feel free to use the Jenkins docker image when practicing remotely, but when deploying for prod, I recommend a bare-metal machine installation - just as will be done below**.

1. It is assumed that you already have an AWS EC2 instance created.

2. Ensure to keep the VM port 8080 open. Jenkins runs by default on port 8080. Keeping everything behind a reverse proxy(e.g. Nginx), plus adding an SSL certified domain or sub-domain(e.g. 'jenkins.mydomain.com'), will also be very beneficial. With that, you get to keep your VM safe - with no need to open the 8080 port. Feel free to refer to my [EC2 Nginx guide](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_nginx-management.md). for handling all of that. 

> One very important thing I also recommend, is that you should always ensure to restrict open ports to the IP of the VM that would be accessing them. Never keep VM ports open to all connections.

- Example Use Case

Allowing port 8080 access only from `198.51.100.10`:

* **Type:** Custom TCP
* **Port Range:** 8080
* **Source:** Change from `0.0.0.0/0` to `198.51.100.10/32` - Custom TCP(IPV4)

## 1.1. Update the system.

```bash
sudo apt update && sudo apt upgrade -y
```

_Screenshot needed._

## 1.2. Install Java.

**Install Java Version 21(a Jenkins requirement) - Ensure to only install version 21 or higher. As at 24/04/2025, version 11 would not work with Jenkins. It has also been announced that Jenkins is already coming to an end of support for version 17.**

```bash
sudo apt install openjdk-21-jre -y # make the installation

java -version # verify the installation
```

_Screenshot needed._

## 1.3 Add Jenkins repo and install(proper)

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

## 1.4 Jenkins system chores - check Jenkins status, and restart if needed.

```bash
sudo systemctl status jenkins
# sudo systemctl start jenkins
sudo systemctl restart jenkins
# sudo systemctl stop jenkins
```

_Screenshot needed._

## 1.5 View the Jenkins logs - in case you need to troubleshoot.

```bash
sudo journalctl -u jenkins.service
```

_Screenshot needed._

## 1.6 Changing the Jenkins port.

**If Jenkins fails to start because a port is in use, run `systemctl edit jenkins` and add the following:**

```bash
[Service]
Environment="JENKINS_PORT=8081"
```

Here, "8081" was chosen but you can put another port available.

_Screenshot needed - show the above(port change) section as shown on this page - https://www.jenkins.io/doc/book/installing/linux/#debianubuntu_

## 1.7 Access Jenkins UI and perform necessary setups.

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

- Log-in, and take a deep breath!!!

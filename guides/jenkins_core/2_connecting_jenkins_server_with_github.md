## Part 2: Setting Up, And Connecting Jenkins With Github(Configuring Github Access).

Jenkins is a CI/CD pipeline creation tool, as such its very important that Jenkins is able to connect(gain access and communicate) with your source code management platform(of course with Github being the most popular). In this guide, I'll show how to connect Jenkins with Github, so that the Jenkins server and Github can communicate via SSH - thus allowing Jenkins to perform actions like pulling code updates.

> **Please note carefully, that this step is different from repository authentication, and web-hook integration for sending push/code-update notifications to Jenkins. All of those will be carefully covered in the relevant guides.**
>
> **All through this guide, ensure to switch to the Jenkins user(on the EC2 VM)**.

```bash
sudo su - jenkins
```

_Screenshot needed._

### 2.1 Generate SSH key(on EC2).

> **Important: Learn more about what(the SSH magic that) is happening in this SSH section of this guide - [Read my free guide on how SSH communication works](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/SSH_communications.md).**

Ensure to copy and save both the public and private key.

```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@ci" -f ~/.ssh/id_rsa_jenkins_controller # creates the keys, abd reveals where to find the public and private keys.
```

_Screenshot needed._

```bash
cat ~/.ssh/id_rsa_jenkins_controller # reveals the private key - never reveal or share this. Except if you wish to delegate the SSH access.
```

_Screenshot needed._


```bash
cat ~/.ssh/id_rsa_jenkins_controller.pub # reveals the public key - copy it.
```

_Screenshot needed._

### 2.2 Add public key to GitHub repo

- Go to your GitHub repo → **Settings → Deploy keys**

> Ensure that the option to add 'Deploy Keys' is enabled/supported for your Github organization.

_Screenshot needed - show how to enable public keys on Github repos._

- Add the public key
- Check **Allow write access** if needed.

My recommendation: leave it unchecked - as an extra layer of safety for you repo. That way, the Jenkins server is only able to read and pull project files.

_Screenshot needed - show how to add the public key - hence properly creating the new deploy key._

### 2.3 Test GitHub access on the EC2 VM(as `jenkins user`)

```bash
sudo su - jenkins # you should have done this by now - as required at the beginning of part 2.
```

```bash
ssh -i ~/.ssh/id_rsa_jenkins_controller git@github.com

# if the configuration is well implemented, this should return a response showing some details of the connected repo.
```

_Screenshot needed._


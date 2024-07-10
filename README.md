# jenkins-pipeline-on-aws

## DevOps CICD pipeline [Jenkins Pipeline] 

_This project will show how to run Jenkins pipeline on AWS [single instance] and run Docker containers using Ansible with github webhook. [For beginner]_

### **Step 1 :** Creating an Instance on AWS.

On AWS EC2 Dashboard click on Launch Instances and Enter the name for instance. Then Select Redhat for Quick Start and Create Key pair, as per shown in Below Figures.

![aws-instance-create-1.1](/../main/Pics/aws-instance-create-1.1.png) <br /><br />
![aws-instance-create-1.2](/../main/Pics/aws-instance-create-1.2.png) <br /><br />

> [!Note]
> select '.pem' format, it will be used as login for instance.

![aws-instance-create-1.3](/../main/Pics/aws-instance-create-1.3.png) <br /><br />
![aws-instance-create-1.4](/../main/Pics/aws-instance-create-1.4.png) <br /><br />
![aws-instance-create-1.5](/../main/Pics/aws-instance-create-1.5.png) <br /><br />

### **Step 2 :** Login to the Instance we just created.

You Need Putty and Puttygen for login for the instance that we just created.

> download link for putti : https://www.putty.org/

After download and installatioin is complete open puttygen we need to convert '.pem' key into '.ppk'

![pem_to_ppk-1.png](/../main/Pics/pem_to_ppk-1.png) <br /><br />

Click on Conversions -> Import key, select '.pem' key and save it as public key.

> [!Caution]
> name the .ppk key same as .pem key.

Now open Putty and select session 

![pem_to_ppk-2.png](/../main/Pics/pem_to_ppk-2.png) <br /><br />

Now enter Public IP of instance and go to Connection -> SSH -> Auth -> Credentials and Browse the .ppk key.

![pem_to_ppk-3.png](/../main/Pics/pem_to_ppk-3.png) <br /><br />

Then Click on Open -> Accept and Enter user name 'ec2-user' [for Red Hat instance]

> [!Note]
> .ppk and .pem are two commonly used file formats for storing public and private keys for secure communication. While .ppk files are specific to PuTTY and use a proprietary format, .pem files use the widely used ASCII text format and can be used to store various types of keys. Despite their differences, both formats serve the same purpose and can be used interchangeably in various cryptographic applications.

### **Step 3 :** Installing Ansible, Jenkins, Docker, and Git on AWS

**Ansible Installation :**

    sudo yum update
    sudo dnf install ansible-core wget

We have to edit the hosts file of Ansible to keep track of all nodes.

    sudo vi /etc/ansible/hosts

Now we have to edit ansible.cfg file 

    sudo vi /etc/ansible/ansible.cfg

Append this on last line of the file: 

    [defaults]
    
    #inventory = /root/anitest.yml
    host_key_checking = False
    #deprecation_warning = False
    #remote_user = root

Thats it for Ansible Installation!!!

**Jenkins Installation :** 

    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
  	sudo yum upgrade
   
    # Add required dependencies for the jenkins package
  	sudo yum install fontconfig java-17-openjdk
  	sudo yum install jenkins
  	sudo systemctl enable jenkins && sudo systemctl start jenkins && sudo systemctl status jenkins

To access the Jenkins webpage which is avilable on port 8080 by default, we need to add port number on inbound rule in security section -> security group -> edit inbound rules

![aws-instance-create-2.png](/../main/Pics/aws-instance-create-2.png) <br /><br />

Now open the Browser and search :-
> http://'Instance IP Address':8080

It will ask you for the CODE, Which you will find on location...

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

![jenkins-askscode](/../main/Pics/jenkins-askscode.png)<br /><br />

After that it will ask Plugins for Jenkins, Just select suggested plugins it will start the installation. Then you will be ask for signup on Jenkins.<br />
Next you will be ask for Login which you just signup for Jenkins. It will take you to Blank Dashboard.

![jenkins-dashboard](/../main/Pics/jenkins-dashboard.png) <br /><br />

Thats it for the Jenkins Installation!!!!

**Docker Installation :**

    sudo dnf check-update    #update the system if need 
    sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io

To start the docker service, 

    sudo systemctl start docker
    sudo systemctl status docker
    sudo systemctl enable docker    #make sure the service is 'active' after system is restarted/reboot 

You need to have Docker Account to Create your own docker images:-

> [!Tip]
> How to create Docker Account link : https://docs.docker.com/docker-id/

How to login in VM using terminal:

    sudo docker login --username "your_email_address"    #Enter the password of docker account

**Git Installation :**

    sudo yum install git 
    sudo git config --global user.email "email_address"
    sudo git config --global user.name "username"

### **Step 4 :** Create a repositorie in GitHub

> how to create new repository
> https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories

We have to Create two files in that Repo. name 'Dockerfile', 'index.html', For example you can use files which is avilable in this Repo.<br />
'Dockerfile' is need for building the images and 'index.html' be our website file.

We need this Repo. in our VM, make directory in '/' name 'git_files' 

    sudo mkdir /git-files
    cd /git-files
    git clone 'URL of your git repo.'

Now add two Files 'docker-build.yml', 'docker-deploy.yml' on location '/git-files/'repository_name/'. This Files will Build image and deploy container respectively. Use the files which is provided in this Repository.

### **Step 5 :**  Creating Pipeline 

Goto Jenkins DashBoard in Browser, Click on 'New item' to create First step of pipeline.
<br />Select Pipeline, Name the project 'aws-pipeline'. <br />Then do the same as shown in below figures:-

![jenkins-pipeline-1.png](/../main/Pics/jenkins-pipeline-1.png) <br /><br />
![jenkins-pipeline-2.png](/../main/Pics/jenkins-pipeline-2.png) <br /><br />
![jenkins-pipeline-3.png](/../main/Pics/jenkins-pipeline-3.png) <br /><br />
![jenkins-pipeline-4.png](/../main/Pics/jenkins-pipeline-4.png) <br /><br />

Save it and Go back to DashBoard and click on New item for Second step of pipline.

Now goto github repository and add new file name 'Jenkinsfile'

Belows are the stages/steps of pipeline 

    #Jenkinsfile
    pipeline{
        agent any
        stages {
            stage('fetch-code'){
                steps {
                    sh '''
                    cd /git-files/jenkins-docker-webpage/
                    sudo git pull
                    '''
                }
            }
            stage('build-image'){
                steps{
                    sh '''
                    cd /git-files/jenkins-docker-webpage/
                    sudo ansible-playbook docker-build.yml
                    '''
                }
            }
            stage('deploy-image'){
                steps{
                    sh '''
                    cd /git-files/jenkins-docker-webpage/
                    sudo ansible-playbook docker-deploy.yml
                    '''
                }
            }
        }
    }

### **Step 6 :** Jenkins-Github Webhook.

First you need to Create token from GitHub
> https://docs.github.com/en/enterprise-server@3.9/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token

In Jenkins : Goto Manage Jenkins -> System Config. -> search for GitHub, click on add Credentials, and then select the Credential that we just created as shown in figure below.

![jenkins-add-github-server.png](/../main/Pics/jenkins-add-github-server.png) <br /><br />

> Use GitHub token as Secret.
![jenkins-add-github-server-2.png](/../main/Pics/jenkins-add-github-server-2.png) <br /><br />

In GitHub : Got to Repository's Setting and click on Webhooks section -> add webhook.

![github-webhook.png](/../main/Pics/github-webhook.png) <br /><br />

### **Step 7 :** Run the Pipeline!!!

Final step... goto dashboard and click on play button of 'aws-pipeline'.

![jenkins-dashboard-1](/../main/Pics/jenkins-dashboard-1.png) <br /><br />

And see the Build Executor Status if its complite or not.

> [!NOTE]
> you might need to refresh the page to check the every stage status. [green or red]

After Build is complite goto VM's Browser and search

> http://'Instance IP Adress':6400

If you see your web page the pipeline was created Susseccfully!!!

it might look like this if you use the provided index.html
![jenkins-webpage](/../main/Pics/jenkins-webpage.png) <br /><br />

<br /><br /><br /><br /><br /><br /><br />_If there is error in pipeline you see the error by clicking on the name which the step is failed and cliking on number of build History -> see console output_

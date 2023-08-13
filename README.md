# Laravel Autodeployment using jenkins

For example we have two server (ubuntu) one is for jenkins and another is for
laravel application server.
1. jenkins@jenkins (For jenkins)
2. app@app (For laravel)

![Servers](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/servers.png?raw=true)

#### Step-1 — Installing Jenkins: 
We will Install jenkins into jenkins@jenkins server

First, we need to install jdk:

``sudo apt install openjdk-11-jdk``

To check java installed successfully:

``java --version``

![Java Version](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/java-version.png?raw=true)

Now you got to add GPG keys:

``sudo wget -p -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -``

![GPG Key](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/add-gpg-key.png?raw=true)

After adding GPG keys, add the Jenkins package address to the sources list by typing the command given below:

``sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/4.png?raw=true)

``sudo apt update``

Let’s move forward and do the real work of installing Jenkins.

``sudo apt install jenkins``

After successfully installation check jenkins is running

``sudo systemctl status jenkins``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/5.png?raw=true)

Also copy the highlighted text password above image and store for future use

#### Step-2 Configure Jenkins Application

Now check your jenkins@jenkins server host

``hostname -I``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/6.png?raw=true)

Now go to your browser and visit ``http://192.168.0.109:8080``

Jenkins runs on ``8080`` port

Now input the previously saved password or ``cat /var/lib/jenkins/secrets/initialAdminPassword``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/8.png?raw=true)

Now click on "Continue"

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/7.png?raw=true)

Now click "Install suggested plugins".

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/9.png?raw=true)

Now installing jenkins plugins. It will take some times. So wait for completing it.

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/10.png?raw=true)

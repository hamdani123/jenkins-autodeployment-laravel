# Laravel Autodeployment using jenkins

For example we have two server (ubuntu) one is for jenkins and another is for
laravel application server.
1. jenkins@jenkins (For jenkins)
2. app@app (For dockerized laravel app)

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

<strong>Now click on "Continue"</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/7.png?raw=true)

<strong>Now click "Install suggested plugins".</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/9.png?raw=true)

<strong>Now installing jenkins plugins. It will take some times. So wait for completing it.</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/10.png?raw=true)

<strong>Now input your information for creating admin user and click "Save and Continue"</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/12.png?raw=true)

<strong>Now input your jenkins URL and click "Save and Finish"</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/13.png?raw=true)

<strong>Jenkins is ready. Now click "Start using jenkins"</strong>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/14.png?raw=true)

Now jenkins is ready for deploying to remote server.

#### Step-3 Make secure remote connection between jenkins server and app server

We need to do this step because, we need to run command remotely from jenkins server.

Now go to jenkins server ssh terminal and type ``ssh-keygen`` hit enter.

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/15.png?raw=true)

Now type ``ssh-copy-id <user>@<ip-address>``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/16.png?raw=true)

Here above, app@192.168.0.110 is user and ip

Now check by running command from jenkins server to app server

``ssh app@192.168.0.110 whoami``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/17.png?raw=true)

Now, Its working perfectly.

#### Step-4 Create jenkins job and pipeline for preparing deployment process

Now go to "New item" on jenkins panel.

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/18.png?raw=true)

Now input item name and select pipeline also click "Ok".

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/19.png?raw=true)

After clicking "Ok", you will get new page then select "Poll SCM" under "Build Triggers"
then scroll down, select "Pipeline script from GCM", "Git" and provide git information under "Pipeline" section.

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/20.png?raw=true)

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/21.png?raw=true)

Follow above procedure. Click "Save".

<b>Note: If you use github or other git provider you can install plugin accordingly. Here, I am using own git server(gitea). I have installed plugins accordingly.</b>

Now jenkins part is done.

#### Step-5 Need to create a "Jenkinsfile" into project root.

Like below

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/22.png?raw=true)

<b>Jenkinsfile file should be like below</b>

````
pipeline {
  agent any
  environment {
    staging_server="192.168.0.110" // app server ip 
    remote_dir="/home/app/rolebase" // app server directory
    remote_user="app" // app server user
  }
  stages {
    stage('Deploy') {
      steps {
        sh 'rsync -avP --exclude ".env" --exclude "vendor" --exclude ".git" --exclude "docker" --exclude="storage" --delete ${WORKSPACE}/ ${remote_user}@${staging_server}:${remote_dir}'
        sh 'scp -r ${WORKSPACE}/docker ${remote_user}@${staging_server}:${remote_dir}'
      }
    }
    stage('Project Build'){
        steps{
            //sh 'ssh ${remote_user}@${staging_server} "rm -rf ${remote_dir}/.editorconfig .env .env.example .idea"'
            sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && cp .env.example .env"'
            sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && docker compose up --build -d"'
            sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && docker compose run --rm app composer install"'
        }
    }
    stage('config:cache'){
        steps{
            sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && docker compose run --rm app php artisan config:cache"'
        }
    }
    stage('Migration') {
      steps {
        sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && docker compose run --rm app php artisan migrate"'
      }
    }

    /* stage('Seed') {
      steps {
        sh 'ssh ${remote_user}@${staging_server} "cd ${remote_dir} && docker compose run --rm app php artisan db:seed"'
      }
    } */
  }
}

````

#### Step-6 Install docker into app server and open relevant port

``sudo ufw allow 80``</br>
``sudo ufw allow 443``</br>
``sudo ufw status``

Follow the instruction form the link: https://docs.docker.com/desktop/install/ubuntu/
<br><center>OR</center></br>

1. ````sudo apt install apt-transport-https ca-certificates curl software-properties-common````
2. ````curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -````
3. ````sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"````
4. ````apt-cache policy docker-ce````
5. ````sudo systemctl status docker````

<b> Note: Also, add app server user to docker group</b>

#### Step-7 Build jenkins job

Click "Build Now" into job details.

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/23.png?raw=true)

Click the highlighted icon

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/24.png?raw=true)

<b>Finally, Deployment successful</b>

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/25.png?raw=true)

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/26.png?raw=true)

First time, it has taken 12.55min for fresh installation.

Note: For .env variables you can manage different directory on jenkins server. Personally I am working on a application which can manage env variable.
Hopefully, I will release it soon.

Now check your application on browser ``http://<app-server-ip-or-domain>``

![alt text](https://github.com/imrancse94/jenkins-autodeployment-laravel/blob/main/27.png?raw=true)

### My application is running 

### !! Thank you !!













# Kubernetes Workshop

## Requirements

1. Ubuntu 16.04 with sudo access
2. [Minikube setup on the machine](https://github.com/gau1991/k8s-workshop/blob/master/readme.md)

## Installation

### Package Update

```bash
sudo apt update
```

### Install Jenkins

1. Set up the repository

    ```bash
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo add-apt-repository "deb https://pkg.jenkins.io/debian-stable binary/"
    sudo apt update
    ```

2. Install Java

    ```bash
    sudo apt install openjdk-8-jdk
    ```

3. Install Jenkins

    ```bash
    sudo apt install jenkins
    ```

## Setup

1. To get the Jenkins initialAdminPassword, run the following command and copy the password

    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

2. Now visit [http://127.0.0.1:8080](http://127.0.0.1:8080)

    ```bash
    sudo minikube start
    ```

    paste the password at the text box and click continue

3. At customize Jenkins Page, click on `Install Suggested Plugins`

4. Once plugin installation is complete, give your Username, Password, Full name and Email. Make sure you note down that password and click on `Save and Continue`

5. For Instance Configuration, just click on Save and Finish. Now you can start using Jenkins by clicking into `Start Using Jenkins`

## First Job Creation

### Docker Hub Sign Up

    **If you already have account to Docker Hub then Skip this Step**

1. Visit to [Docker Hub](https://hub.docker.com/) and click on `Sign In`.

2. Now click on `Sign up` and finish the the Docker Hub Sign Up Process

### Github Sign Up

    **If you already have account to Github then Skip this Step**

1. Visit to [Github](https://github.com/) and click on Sign Up.

2. Now finish the the Github Sign Up Process

### Create Docker Hub Repository

1. Visit to the [Docker Hub Repository Creation](https://hub.docker.com/repository/create) page.

2. In Name field, give Name as `sample-demo-app`

3. For Visibility, click on `Public`

4. Now at the end click  on `Create`

### Fork Github Project

1. Now visit to the [jenkins-workshop](https://github.com/gau1991/jenkins-workshop) and on top corner click on `Fork` icon

2. Finish the fork process.

### Updated Jenkinsfile

1. Now goto forked repo and click on the `Jenkinsfile`. On the `Edit thie File` icon.

2. At line no. `7`, replace URL with your forked repo URL

3. At line no. `25, 26 and 32`, replace `gau1991` with your Docker Hub login id.

4. At bottom of page click on `Commit Changes`

### Pre Job Setup

1. Now go to command prompt and check whether minikube is running or not

    ```bash
    sudo minikube status
    ```

    if minikube is not running, then start it with

    ```bash
    sudo minikube start
    ```

2. Install the required packages

    ```bash
    sudo apt update
    sudo apt install python3 python3-pip git
    sudo pip3 install django pylint
    ```

3. Now copy kubectl config to Jenkins home directory

    ```bash
    sudo cp -r $HOME/.minikube $HOME/.kube /var/lib/jenkins/
    sudo chown -R jenkins: /var/lib/jenkins
    ```

4. Add Jenkins user to docker group and restart the Jenkins

    ```bash
    sudo usermod -aG docker jenkins
    sudo service jenkins restart
    ```

5. Now open the file `/var/lib/jenkins/.kube/config` in editor and replace all the `/home/ubuntu` with `/var/lib/jenkins`

    ```bash
    sudo vim /var/lib/jenkins/.kube/config
    ```

6. Switch to jenkins user

    ```bash
    sudo su - jenkins
    ```

7. Run the kubectl command to see whether kubectl is connecting to minikube

    ```bash
    kubectl get pods -A
    docker ps
    ```

8. At the end, login to Docker Hub using following command

    ```bash
    docker login
    ```

### Job Creation

1. Login to Jenkins and click on `create new jobs`

2. Give name as `ci-cd` and click `pipeline` project. Now click on `Ok`

3. In configuration page, for `general` section select `GitHub project` and in project url field give your forked github project url

4. In `Build Triggers` section, select `Poll SCM` checkbox and in text area give `H/5 * * * *`

5. In `Pipeline` section, for `Definition` select `Pipeline Script from SCM`. For `SCM`, select `Git` from dropdown. For `Repository URL` section, give your forked repo url.

6. Click on `Save`

### Launch the deployment

1. Job will fail beacause we don't have deployment running for my server. So to create deployment use following command

    ```bash
    kubectl create deployment --image gau1991/sample-demo-app:1 myserver
    kubectl expose deployment myserver --type=NodePort --port=8000
    ```

2. Now retrigger job from Jenkins by click `Build Now` Button

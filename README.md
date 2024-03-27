# JenkinsDockerDeploy
Deploy applications to a Docker server effortlessly with Jenkins automation. Simplify your deployment process and enhance productivity with this comprehensive guide.

# Deploy to Docker Server

This guide outlines the steps to deploy a project to a Docker server using Jenkins automation.

## Step 1: Setup Jenkins Server

- Launch an EC2 instance with Amazon Linux 2 AMI.
- Open TCP ports 8080 (Jenkins UI) and 22 (SSH) in the Security Group.
- Install Java:
  ```bash
  sudo amazon-linux-extras install java-openjdk11
  ```
- Install Jenkins:
  ```bash
  sudo yum install jenkins
  ```
- Start and enable Jenkins service:
  ```bash
  sudo systemctl start jenkins
  sudo systemctl enable jenkins
  ```
- (Optional) Set a hostname for the server:
  ```bash
  sudo vi /etc/hostname
  ```
  Enter `jenkins-server` in the file.

## Step 2: Integrate Git with Jenkins

- Install Git on Jenkins server:
  ```bash
  sudo yum install git
  ```
- Install GitHub plugin on Jenkins UI.

## Step 3: Integrate Maven with Jenkins

- Navigate to the `/opt` directory:
  ```bash
  cd /opt
  ```
- Download and extract Apache Maven.
- Set up environment variables for Java and Maven in `~/.bash_profile`.
- Install Maven Integration plugin on Jenkins UI.
- Configure Maven in Jenkins.

## Step 4: Setup Docker Server

- Launch another EC2 instance with the same Security Group and AMI.
- Install Docker:
  ```bash
  sudo yum install docker
  ```
- Start and enable Docker service:
  ```bash
  sudo systemctl start docker
  sudo systemctl enable docker
  ```
- Configure permissions for Dockerfile and Docker directory:
  ```bash
  sudo mkdir /opt/docker
  sudo chown -R <username>:<group> /opt/docker
  ```
- Create a Dockerfile in `/opt/docker` with appropriate contents:
  ```Dockerfile
  FROM Tomcat
  RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
  COPY webapp/target/*.war /usr/local/tomcat/webapps
  ```


## Step 5: Integrate Docker with Jenkins

- Create a new user for accessing Docker:
  ```bash
  sudo useradd dockeradmin
  sudo passwd dockeradmin
  sudo usermod -aG docker dockeradmin
  ```
- Allow the new user to be connected using PasswordAuthentication:
  ```bash
  sudo vi /etc/ssh/sshd_config
  ```
  Change `PasswordAuthentication no` to `PasswordAuthentication yes`.
- Reload the SSH service:
  ```bash
  sudo service sshd reload
  ```
- Install "Publish over SSH" plugin on Jenkins UI.
- Configure SSH server for Docker instance in Jenkins:
  - Manage Jenkins > System > Configure System
  - Scroll down to "Publish over SSH" section
  - Click on "SSH Servers" and then "Add"
    - Name: docker
    - Hostname: <docker-instance-ip>
    - Username: dockeradmin
    - Advanced > Use password authentication
    - Enter password
- Create a Docker directory in `/opt` and set appropriate permissions:
  ```bash
  sudo mkdir /opt/docker
  sudo chown -R dockeradmin:dockeradmin /opt/docker
  ```

## Step 6: Configure the Jenkins Job

- Use Git as SCM and set the repository URL.
- Set up polling SCM.
- Configure build goals and options.
- Add post-build actions to send build artifacts over SSH to Docker server:
  - Name: docker
  - SourceFiles: **/*.war
  - Remote directory: /opt/docker
  - Exec command:
    ```bash
    cd /opt/docker;
    docker build -t regimage:v1 .;
    docker stop regapp;  
    docker rm regapp;
    docker run -d --name regapp -p 8081:8080 regimage:v1
    ```

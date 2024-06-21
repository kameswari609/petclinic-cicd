# Containerizing a Java Application and Implementing a CI/CD Pipeline

<details>
  <summary>Architecture</summary>
  <img src="./Images/two.png">
</details>

<details>
<summary>Steps followed to create the project</summary>
<br>

<details class="nested">
<summary>Create GitHub repository</summary><br>
We will create a GitHub repository for Part One. This repository will serve as a central hub for developers to easily interact with the               project and manage their contributions. It will also provide a solid anchor for our Jenkins pipeline, ensuring smooth integration and                 continuous deployment processes. By utilizing GitHub, we promote collaboration, version control, and transparency within the team,                    enhancing overall productivity and project management.<br><br> 
clone this repo: https://github.com/spring-projects/spring-petclinic
</details>
   

<details class="nested">
<summary>Setup the Sonarcloud</summary><br>
1. Go to sonarcloud.io<br><br>  
2. Login with GitHub<br><br>  
3. Create a new Organisation<br><br>
  <img src="./Images/sonarcloud1.png">
  Then click on Create Organisation <br>
4. Create a Project.<br>
  <img src="./Images/sonarcloud2.png"><br>
  Follow the setup and then click on Create Project.
5.
</details>

<details class="nested">
<summary>Setup the Jfrog</summary><br>
1. Go to Jfrog.io<br><br>  
2. Login with Username and Password<br>
   make sure to save the username and password as we will need them as credentials in future steps.<br>  
3. Go to Administration on the main page>>Repositories>>create repository>>local>>docker>>repositoryname>>click on create repo<br>
   <img src="./Images/jfrog1.png"><br>
4. Copy the URL and click on set up docker client>> Give the token name and follow the steps in the ec2 created.<br>
   <img src="./Images/jfrog2.png"><br>
5. Create a Project.<br>
</details>


<details class="nested">
<summary>Launch an ec2</summary><br>
We will create an EC2 instance on AWS and set up the project there. This way, the project setup won’t interfere with our local machines, and our      local setups won’t affect the project. By isolating the environment, we ensure a clean and consistent setup for everyone involved, making it easier to manage dependencies and configurations. Additionally, this approach allows for better scalability and flexibility as we can easily         replicate the environment or scale resources as needed.<br><br>

1. Install Jenkins on the Instance.<br><br>
<pre><code>  
sudo apt update
sudo apt install openjdk-11-jdk
java --version
wget -p -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl status jenkins
sudo systemctl start jenkins
</code> </pre><br>
2. Install Maven on the Instance.<br><br>
<pre><code> 
$ wget https://mirrors.estointernet.in/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
$ tar -xvf apache-maven-3.6.3-bin.tar.gz
$ mv apache-maven-3.6.3 /opt/
M2_HOME='/opt/apache-maven-3.6.3'
PATH="$M2_HOME/bin:$PATH"
export PATH
$ mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /opt/apache-maven-3.6.3
Java version: 13.0.1, vendor: Oracle Corporation, runtime: /opt/jdk-13.0.1
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "4.15.0-47-generic", arch: "amd64", family: "unix"
</code> </pre><br>
</details>

<details class="nested">
<summary>Integrating maven with Jenkins</summary><br>
1. Go to Jenkins Dashboard >> Manage Jenkins >> tools >> Maven Installation >> Add Maven<br>
2. either opt for install automatically or give the name and path of the maven in your ec2 for reference which could later be called in the Jenkins pipeline.
  <img src="./Images/maven1.png"><br>
</details>

<details class="nested">
<summary>Integrating SonarCloud with Jenkins</summary><br>
1. Go to SonarCloud Dashboard >> Account ID >> My Account >> Security >> Generatetoken<br>
2. Copy the code and paste it into the pipeline.
   <img src="./Images/sonarcloud3.png"><br>
</details>

<details class="nested">
<summary>Integrating Github with Jenkins with the help of webhooks</summary><br>
1. Go to GitHub Dashboard >> Repository >> Settings >> webhook<br>
2. fill the payload URL as https://localhost:portnumber/jenkins-webhook/ , content type<br>
3. Click on Update webhook<br>
 <img src="./Images/webhook1.png"><br>

**Note:** We have used Ngrok to serve as a server proxy as my instance is running in localhost and is not accessible by the internet.
  
</details>


<details class="nested">
<summary>Jenkins File</summary><br>
<p>Path to the Jenkinsfile file: <a href = "./Jenkinsfile"> Jenkinsfile</a></p>
</details>

  <details class="nested">
  <summary>Launch Jenkins and Create a Pipeline with the following stages</summary><br>
  <p>Path to the Provisioner file: <a href = "./Jenkinsfile"> Jenkinsfile</a></p>
  Steps followed:<br>
    1. Create a Jenkins job with the following configurations:<br><br>
      &nbsp;a. Item Type: Multibranchpipeline<br>
      &nbsp;b. Branch Source: Git, give your git repo URL, Property strategy: all branches get the same properties, Build Configuration: 
      Jenkinsfile, 
      Scan Multibranch Pipeline Trigger: scan by webhook, give the trigger token and follow the steps in github<br><br>
    2. Install the following plugins: GitHub Plugin, GitHub Branch Source Plugin,Pipeline Plugin, Multibranch Pipeline Plugin, SonarQube Scanner 
    Plugin, JFrog Artifactory Plugin, Docker Plugin, Docker Pipeline Plugin:<br><br>
    3. Creta a credential in Jenkins to store the  AWS access key and secret key as username and password also pass the ID, similarly for docker and 
   jfrog <br><br>
    4. Build the pipeline if no errors occur we will be able to find an ami in the location specified.
    <img src="./Images/jenkins1.png">
  </details>

  <details class="nested">
  <summary>Check for the Docker image in Docker Hub as well as Jfrog Consoles</summary><br>
     we will be able to find the docker image with the branch and commit ID as a tag.
    <img src="./Images/dockerhub1.png"><br>
    <img src="./Images/jfrog3.png"><br>
    
  </details>
 
  </details>

Reference to the other parts of the Parts.
|[Part 0](https://github.com/AnirudhBadoni/ProjectOne.git)|[Part 1](https://github.com/AnirudhBadoni/Packer.git)|[Part 3](https://github.com/AnirudhBadoni/AwsInfra.git)|
|---|---|---|

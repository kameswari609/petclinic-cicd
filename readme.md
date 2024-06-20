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
<summary> Scan code with Sonarcloud</summary><br>
1. Go to sonarcloud.io<br><br>  
2. Login with GitHub<br><br>  
3. Create an Organisation<br><br>
4. Create a Project.
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
<summary></summary><br>
 <p>Path to the Provisioner file: <a href = "./aws-ami-v1.pkr.hcl"> aws-ami-v1.pkr.hcl</a></p>
</details>

  <details class="nested">
  <summary>Provisioner</summary><br>
  <p>Path to the Provisioner file: <a href = "./provisioner.sh"> Provisioner.sh</a></p>
  </details>

  <details class="nested">
  <summary>Launch Jenkins and Create a Pipeline with the following stages</summary><br>
  <p>Path to the Provisioner file: <a href = "./Jenkinsfile"> Jenkinsfile</a></p>
  Steps followed:<br>
    1. Create a Jenkins job with the following configurations:<br><br>
      &nbsp;a. Item Type: pipeline<br>
      &nbsp;b. Pipeline Definition(at the last of the configuration): Pipeline Script from SCM, SCM: Git, Repository URL: your repository URL,             Credentials: &nbsp; none, Branc Specifier(main/master whichever is yours), Script Path: Jenkinsfile.<br><br>
    2. Install the following plugins: Git Plugin, GitHub Integration Plugin, Pipeline: GitHub Plugin, AWS Credentials Plugin, and AWS Steps Plugin.<br><br>
    3. Creta a credential in Jenkins to store the  AWS access key and secret key as username and password also pass the ID.<br><br>
    4. Build the pipeline if no errors occur we will be able to find an ami in the location specified.
    <img src="./Images/jenkins1.png">
  </details>

  <details class="nested">
  <summary>Check for the ami on AWS Console</summary><br>
     We will be able to find the ami with the name and tags passed by us in the region specified by us in the AWS console.
    <img src="./Images/aws1.png">
    
  </details>
 
  </details>

Reference to the other parts of the Parts.
|[Part 0](https://github.com/AnirudhBadoni/ProjectOne.git)|[Part 1](https://github.com/AnirudhBadoni/Packer.git)|[Part 3](https://github.com/AnirudhBadoni/AwsInfra.git)|
|---|---|---|

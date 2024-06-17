pipeline {
    agent any
    tools {
        maven 'maven_3_6_3'
    }
    
    stages {
        stage('Prepration'){
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                script{
                     // Checkout code from GitHub
                    def branchName = env.BRANCH_NAME
                    echo "Checking out code from branch: ${branchName}"
                    checkout([$class: 'GitSCM', 
                    branches: [[name: "${branchName}"]],  // Fetch code from all branches
                    userRemoteConfigs: [[url: 'https://github.com/AnirudhBadoni/Petclinic.git']]]) 

                    // Get the latest commit hash
                    def commitId = sh(script: 'git rev-parse HEAD', returnStdout: true).trim().take(7)
                    env.IMAGE_TAG = "${branchName}-${commitId}"
                    echo "Docker image tag: ${env.IMAGE_TAG}"
                }
            }
        }
        
        stage('Build') {
            steps {
                // Your build steps here
                sh './mvnw package'
               
            }
        }
        
        stage('Sonar Scan') {
            steps {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=Petclinic -Dsonar.organization=anirudhbadoni -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=b63dd391f27bf876bed3136541a749490a938243'
                
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("anirudhbadoni/petclinic:${env.IMAGE_TAG}")
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            environment {
                DOCKER_REGISTRY = 'https://index.docker.io/v1/'
                DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
            }
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${env.IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Push Docker image to Jfrog'){
            steps {
                script {
                    echo 'Pushing to Jfrog'
                    withCredentials([UsernamePassword](credentialsId: jfrog-credentials, passwordVariable: 'PASS', usernameVariable: 'USER')){
                        sh "anirudhbadoni/petclinic:${env.IMAGE_TAG} anirudhbadoni.jfrog.io/artifactory/api/docker/anirudh-local/anirudhbadoni/petclinic:${env.IMAGE_TAG}"
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push anirudhbadoni.jfrog.io/artifactory/api/docker/anirudh-local/anirudhbadoni/petclinic:${env.IMAGE_TAG}"
                    }
                }
            }
        }
    }
}
    


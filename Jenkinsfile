pipeline {
    agent any
    tools {
        maven 'maven_3_6_3'
    }
    environment {
        ARTIFACTORY_CREDENTIALS = credentials('jfrog-credentials')  // Use credentials plugin to handle Artifactory credentials
        ARTIFACTORY_URL = 'anirudhbadoni.jfrog.io'
        ARTIFACTORY_DOCKER_REPO = 'docker-local'
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

        stage('Push Docker Image to JFrog Artifactory') {
            steps {
                script {
                    // Login to JFrog Artifactory Docker registry
                    sh "docker login -u ${ARTIFACTORY_CREDENTIALS_USR} -p ${ARTIFACTORY_CREDENTIALS_PSW} ${ARTIFACTORY_URL}"

                    // Tag the Docker image
                    sh "docker tag anirudhbadoni/petclinic:${env.IMAGE_TAG} ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/anirudhbadoni/petclinic:${env.IMAGE_TAG}"

                    // Push the Docker image to JFrog Artifactory
                    sh "docker push ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/anirudhbadoni/petclinic:${env.IMAGE_TAG}"
                }
            }
        }
    }
}
    


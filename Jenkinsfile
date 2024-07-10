pipeline {
    agent any
    tools {
        maven 'maven_3_8_7'
    }
    environment {
        ARTIFACTORY_CREDENTIALS = credentials('jfrog-credentials')  // Use credentials plugin to handle Artifactory credentials
        ARTIFACTORY_URL = 'aparnamk.jfrog.io'
        ARTIFACTORY_DOCKER_REPO = 'aparna'
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
                    userRemoteConfigs: [[url: 'https://github.com/kameswari609/petclinic-cicd.git']]]) 

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
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=packerkey -Dsonar.organization=packeraws -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=20157908dffc0c41730e79135cb0fd1aff632cc1'
                
            }
        }
        
       stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "aparnamantravadi/petclinicdeploy:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'cd petclinic-cicd && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v2/', "docker-cred") {
                dockerImage.push()
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
    


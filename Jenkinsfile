pipeline {
    agent any
   
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
        sh 'echo passed'
        
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
         withMaven {
          sh "mvn clean verify"
        } 
          // build the project and create a JAR file
          // sh './mvnw package'
          //sh 'mvn package'
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
                    sh "docker tag aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG} ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG}"

                    // Push the Docker image to JFrog Artifactory
                    sh "docker push ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG}"
                }
            }
        }
    }
}
    


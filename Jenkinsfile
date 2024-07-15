pipeline {
    agent any
   
   
    stages {
        stage('Prepration'){
            steps {
                cleanWs()
            }
        }
        

    stage('Checkout') {
      steps {
        sh 'echo passed the git checkout'
        git branch: 'main', url: 'https://github.com/kameswari609/petclinic-cicd.git'
      }
    }


            
        
        
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
          
          // build the project and create a JAR file
              sh'pwd'
            // sh 'mvn - f  /petclinic-cicd/blob/main/pom.xml clean test'
           //sh 'mvn -B -DskipTests clean package'
              sh './mvnw package'
          //sh 'mvn package'
      }
    }
        
        stage('Sonar Scan') {
            steps {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=packerkey_petclinicdeployproject -Dsonar.organization=packerkey -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=20157908dffc0c41730e79135cb0fd1aff632cc1'
                // mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=packerkey_petclinicdeployproject
                

            }
        }
        
       stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("kameswari609/petclinic-cicd:${env.IMAGE_TAG}")
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
                    sh "docker tag aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG} ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG}"

                    // Push the Docker image to JFrog Artifactory
                    sh "docker push ${ARTIFACTORY_URL}/${ARTIFACTORY_DOCKER_REPO}/aparnamantravadi/petclinicdeploy:${env.IMAGE_TAG}"
                }
            }
        }
    }
}
    


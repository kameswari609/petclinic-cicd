pipeline {
    agent any

    tools {
        maven 'Maven 3.x'  // Assuming Maven is configured in Jenkins
        jdk 'JDK 11'       // Assuming JDK is configured in Jenkins
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Assuming a SonarCloud token is stored in Jenkins credentials
        ARTIFACTORY_USER = credentials('artifactory-username')
        ARTIFACTORY_PASSWORD = credentials('artifactory-password')
        ARTIFACTORY_URL = 'https://your-artifactory-instance/artifactory'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        if (fileExists('pom.xml')) {
                            sh 'mvn sonar:sonar -Dsonar.projectKey=your_project_key -Dsonar.organization=your_org -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=$SONAR_TOKEN'
                        } else {
                            echo "No pom.xml found, skipping SonarCloud analysis."
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('pom.xml')) {
                        sh 'mvn clean package'
                    } else {
                        echo "No pom.xml found, skipping Maven build."
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            when {
                expression {
                    return fileExists('Dockerfile') && env.BRANCH_NAME ==~ /(main|master)/
                }
            }
            steps {
                script {
                    def dockerImage = docker.build("your-image-name:${env.BRANCH_NAME}-${env.BUILD_ID}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    def server = Artifactory.newServer url: ARTIFACTORY_URL, username: ARTIFACTORY_USER, password: ARTIFACTORY_PASSWORD
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "your-repo-path/${env.BRANCH_NAME}/"
                            },
                            {
                                "pattern": "target/*.war",
                                "target": "your-repo-path/${env.BRANCH_NAME}/"
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec
                }
            }
        }

        stage('Tagging') {
            steps {
                script {
                    def tagName = env.BRANCH_NAME == 'main' ? 'stable' : env.BRANCH_NAME
                    sh "git tag -a ${tagName}-${env.BUILD_ID} -m 'Build ${env.BUILD_ID}'"
                    sh "git push origin ${tagName}-${env.BUILD_ID}"
                }
            }
        }
    }

    post {
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}

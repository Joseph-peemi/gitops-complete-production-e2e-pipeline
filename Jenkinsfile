pipeline {
    agent {
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-production-e2e-pipeline"
        RELEASE_VERSION = "1.0.0"
        DOCKER_USER = "abuchijoe"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/Joseph-peemi/complete-prodcution-e2e-pipeline'
            }
        }
        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t ${IMAGE_NAME}:${RELEASE_VERSION} ."
                        sh "docker tag ${IMAGE_NAME}:${RELEASE_VERSION} ${IMAGE_NAME}:latest"
                        sh "docker push ${IMAGE_NAME}:${RELEASE_VERSION}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        stage("Checkout GitOps Repo") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline'
            }
        }
        stage("Update Deployment File") {
            steps {
                script {
                    sh """
                        cat deployment.yaml
                        sed -i 's|${APP_NAME}:.*|${APP_NAME}:${RELEASE_VERSION}|g' deployment.yaml
                        cat deployment.yaml
                    """
                }
            }
        }
        stage("Push to Git Repository") {
            steps {
                script {
                    sh """
                        git config user.name "Abuchi Joe"
                        git config user.email "peemijoe9522@gmail.com"
                        git add deployment.yaml
                        git commit -m "Updated deployment image to version ${RELEASE_VERSION}"
                    """
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        sh "git push https://github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline HEAD:main"
                    }
                }
            }
        }
    }
}

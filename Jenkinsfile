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
        DOCKER_USER = "abuchijoe"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        GIT_USER = "Joseph-peemi"
        GIT_CREDENTIALS = "github"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout GitOps Repo") {
            steps {
                git branch: 'main',
                    credentialsId: env.GIT_CREDENTIALS,
                    url: 'https://github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline'
            }
        }
        stage("Update Deployment File") {
            steps {
                script {
                    sh """
                        sed -i 's|image: ${APP_NAME}:.*|image: ${IMAGE_NAME}:${RELEASE_VERSION}|g' deployment.yaml
                    """
                }
            }
        }
        stage("Push Deployment Update") {
            steps {
                script {
                    sh """
                        git config user.name "${GIT_USER}"
                        git config user.email "peemijoe9522@gmail.com"
                        git add deployment.yaml
                        git commit -m "Updated deployment image to version ${RELEASE_VERSION}" || echo "No changes to commit"
                    """
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh "git push https://${GIT_USER}:${GIT_PASS}@github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline.git HEAD:main"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
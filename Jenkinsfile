pipeline {
    agent { label "jenkins-agent" }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'abuchijoe/complete-production-e2e-pipeline:latest', description: 'Docker image tag to deploy')
    }
    environment {
        APP_NAME = "complete-production-e2e-pipeline"
        GIT_CREDENTIALS = "github"
        GIT_USER = "Joseph-peemi"
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    stages {
        stage("Cleanup Workspace") {
            steps { cleanWs() }
        }
        stage("Checkout GitOps Repo") {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline.git',
                        credentialsId: env.GIT_CREDENTIALS
                    ]]
                ])
            }
        }
        stage("Update Deployment File") {
            steps {
                script {
                    sh """
                        sed -i 's|image: ${APP_NAME}:.*|image: ${params.IMAGE_TAG}|g' deployment.yaml
                        cat deployment.yaml   # Optional: show the change for debugging
                    """
                }
            }
        }
        stage("Push Deployment Update") {
            steps {
                script {
                    sh """
                        git config user.name "Joseph-peemi"
                        git config user.email "peemijoe9522@gmail.com"
                        git add deployment.yaml
                        git commit -m "Updated deployment image to ${params.IMAGE_TAG}" || echo "No changes to commit"
                    """
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Joseph-peemi/gitops-complete-production-e2e-pipeline.git HEAD:main'''
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
pipeline {
    agent {
        label "jenkins-agent"
    }
    environment {
        APP_NAME = "complete-production-e2e-pipeline"
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

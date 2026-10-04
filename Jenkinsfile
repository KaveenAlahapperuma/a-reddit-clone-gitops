pipeline {
    agent any
    environment {
          APP_NAME = "reddit-clone-app"
    }
    stages {
         stage("Cleanup Workspace") {
             steps {
                cleanWs()
             }
         }
         stage("Checkout from SCM") {
             steps {
                     git branch: 'main', credentialsId: 'github', url: 'https://github.com/KaveenAlahapperuma/a-reddit-clone-gitops'
             }
         }
         stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat deployment.yaml
                    sed -i 's|image: kaveend4/${APP_NAME}:.*|image: kaveend4/${APP_NAME}:${IMAGE_TAG}|' deployment.yaml
                    cat deployment.yaml
                """
            }
         }
         stage("Push the changed deployment file to GitHub") {
            steps {
                sh """
                    git config --global user.name "KaveenAlahapperuma"
                    git config --global user.email "kaveendinuka040@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/KaveenAlahapperuma/a-reddit-clone-gitops main"
                }
            }
         }
    }
}

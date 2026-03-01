pipeline {
    agent any

    options {
         disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME= "kasbesruthi/project-1"
        GIT_USER= "kasbesruthi"
        GIT_EMAIL= "kasbesruthi73@gmail.com"
    }

    stages {
        stage ('git checkout') {
            steps {
                checkout scm
            }
        }
        stage ('Build and Push') {
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: "dockerhub-creds",
                        usernameVariable: "DOCKER_USER",
                        passwordVariable: "DOCKER_PASS"
                    )]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage ('Update k8s manifest') {
            when { branch 'main' }
            steps {
                script{
                    withCredentials([usernamePassword(
                        credentialsId: "git-creds",
                        usernameVariable: "GIT_USERNAME",
                        passwordVariable: "GIT_TOKEN"
                    )]) {
                        sh """
                            set -e
                            git config user.name "$GIT_USER"
                            git config user.email "$GIT_EMAIL"

                            git fetch origin
                            git checkout main
                            git reset --hard origin/main

                            sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                            git add k8s/deployment.yml
                            git diff --cached --quiet || git commit -m "Manifest file updated with ${IMAGE_TAG}"
                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/${GIT_USER}/project-1.git main
                        """
                    }
                }
            }
        }
    }
}
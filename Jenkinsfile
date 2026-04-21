pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/adamski2004/DevOps_A2_Repo_A.git'
        REPO_BRANCH = 'main'
        DOCKER_HUB_USERNAME = 'adamski2004'
        IMAGE_NAME = "${DOCKER_HUB_USERNAME}/runcalc-pro"
        VERSION = "v1.0.${BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${REPO_BRANCH}",
                    url: "${REPO_URL}"
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$VERSION .
                '''
            }
        }

        stage('Tag Latest') {
            steps {
                sh '''
                    docker tag $IMAGE_NAME:$VERSION $IMAGE_NAME:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$VERSION
                        docker push $IMAGE_NAME:latest
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Successfully pushed ${IMAGE_NAME}:${VERSION}"
        }
        failure {
            echo "CI pipeline failed"
        }
    }
}
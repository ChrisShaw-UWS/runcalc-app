pipeline {
    agent any

    environment {
        DOCKER_REPO = "chriss/runcalc-pro"
        VERSION = "v1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                sh "docker build -t $DOCKER_REPO:$VERSION ."
            }
        }

        stage('Tag Latest') {
            steps {
                echo "Tagging Image..."
                sh "docker tag $DOCKER_REPO:$VERSION $DOCKER_REPO:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing to Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_REPO:$VERSION
                        docker push $DOCKER_REPO:latest
                    """
                }
            }
        }
    }
}
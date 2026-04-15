pipeline {
    agent any

    environment {
        DOCKER_REPO = "uwschriss/runcalc-pro"
        VERSION = "v1.0.${BUILD_NUMBER}"
        EC2_HOST = "54.88.196.120"
        CONTAINER_NAME = "runcalc"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                sh "docker build -t $DOCKER_REPO:$VERSION ."
            }
        }

        stage('Tag Latest') {
            steps {
                echo "Tagging latest..."
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

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $DOCKER_REPO:$VERSION
                    docker push $DOCKER_REPO:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
    steps {
        echo "Deploying to EC2..."

    sh '''
    ssh -i /var/lib/jenkins/RunCalcPro.pem ubuntu@54.88.196.120 \\
    'docker pull uwschriss/runcalc-pro:latest && \\
    docker stop runcalc || true && \\
    docker rm runcalc || true && \\
    docker run -d -p 80:80 --name runcalc uwschriss/runcalc-pro:latest'
    '''
    }
}

        stage('Health Check') {
            steps {
                echo "Running health check..."

                sh '''
                sleep 10
                curl -f http://54.88.196.120 || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Application is live (thank goodness)."
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details (again)."
        }
    }
}
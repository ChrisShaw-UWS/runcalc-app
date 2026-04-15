pipeline {
    agent any

    environment {
        DOCKER_REPO = "uwschriss/runcalc-pro"
        VERSION = "v1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_REPO:$VERSION ."
            }
        }

        stage('Tag Latest') {
            steps {
                sh "docker tag $DOCKER_REPO:$VERSION $DOCKER_REPO:latest"
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
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_REPO:$VERSION
                        docker push $DOCKER_REPO:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -i /var/lib/jenkins/RunCalcPro.pem ubuntu@54.88.196.120 << EOF
                docker pull uwschriss/runcalc-pro:latest
                docker stop runcalc || true
                docker rm runcalc || true
                docker run -d -p 80:80 --name runcalc uwschriss/runcalc-pro:latest
                '
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10
                    curl -f http://54.88.196.120 || exit 1
                '''
            }
        }
    }
}
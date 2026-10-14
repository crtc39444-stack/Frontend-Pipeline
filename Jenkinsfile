
pipeline {
    agent any

    environment {
        IMAGE = 'goutam2705/frontend-pipeline'
        EC2_HOST = '15.206.174.3'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'goutam2705',
                        passwordVariable: 'priyal chinaray'
                    )
                ]) {
                    sh '''
                        docker build -t $IMAGE:$BUILD_NUMBER .
                        echo "$TOKEN" | docker login -u "$USER" --password-stdin
                        docker push $IMAGE:$BUILD_NUMBER
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        ssh ubuntu@$EC2_HOST "
                            sudo docker pull $IMAGE:$BUILD_NUMBER &&
                            (sudo docker rm -f frontend-app || true) &&
                            sudo docker run -d --name frontend-app \
                                --restart unless-stopped \
                                -p 80:80 $IMAGE:$BUILD_NUMBER
                        "
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh 'curl -f http://15.206.174.3/'
            }
        }
    }
}

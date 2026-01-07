pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_URI = '355646525228.dkr.ecr.ap-south-1.amazonaws.com/anilist_image'
        IMAGE_TAG = '1.0'
        REPO_NAME = 'anilist_image'
    }
    stages {
        stage('checkout scm') {
            steps {
                checkout scm
            }
        }
        stage('building docker image') {
            steps {
                sh "docker build -t ${ECR_URI}:${IMAGE_TAG} ."
            }
        }
        stage('push to ECR') {
            steps {
                withAWS(credentials: 'aws_credentials', region: AWS_REGION) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                        docker push ${ECR_URI}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('deploy on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh', usernameVariable: 'SSH_USER', keyFileVariable: 'SSH_KEY')]) {
                    withAWS(credentials: 'aws_credentials', region: AWS_REGION) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@65.0.176.60 << EOF
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                    docker pull ${ECR_URI}:${IMAGE_TAG}
                    docker rm -f anilist-container || true
                    docker run -d --name anilist-container -p 80:80 ${ECR_URI}:${IMAGE_TAG}
                    EOF
                    '''
                    }
                }
            }
        }
    }
}
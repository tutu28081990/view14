pipeline {
    agent any

    environment {
        IMAGE = 'mj2808/nodedemo01:latest'
        DEPLOY_HOST = '172.31.0.203'
        CONTAINER_NAME = 'nodedemo01'
    }

    tools {
        nodejs 'node24'
    }
    stages {
        stage ("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main',
                    credentialsId: '44f9abca-2802-4aeb-b7cf-e323982435af',
                    url: 'https://github.com/tutu28081990/view14.git'
            }
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage("Build Docker Image") {
            steps {
                sh "docker build -t nodedemo01 ."
            }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag nodedemo01 mj2808/nodedemo01:latest"
                        sh "docker push mj2808/nodedemo01:latest"
                    }
                }
            }
        }
        stage("Deploy to EC2") {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'nodemoec2',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" "$SSH_USER@$DEPLOY_HOST" "
                            set -e
                            docker pull $IMAGE
                            docker rm -f $CONTAINER_NAME || true
                            docker run -d --name $CONTAINER_NAME --restart unless-stopped -p 80:3000 $IMAGE
                        "
                    '''
                }
            }
        }
    }
}

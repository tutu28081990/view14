pipeline {
    agent any
    tools {
        jdk 'jdk17'
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
                sh "docker build -t starbucks ."
            }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag starbucks vikas4cloud/starbucks:latest"
                        sh "docker push vikas4cloud/starbucks:latest"
                    }
                }
            }
        }
    }
}

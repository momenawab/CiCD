pipeline {
    agent any
   
    environment {
        registry = "docker.io"
        reponame = "believeer"
        appname = "ci-cd"
        env = "prod"
    }
    
    stages {
        // Build Stage
        stage('build') {
            steps {
                sh """
                    docker build -t $registry/$reponame/$appname:$BUILD_NUMBER .
                    docker images
                """
            }
        }

        stage('push Image') {
            when {
                environment name: 'env', value: 'prod'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin docker.io
                        docker push $registry/$reponame/$appname:$BUILD_NUMBER
                    """
                }
            }
        }

        // Deploy stage 
        stage("deploy to ec2") {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sshagent(credentials: ['ec2-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ec2-user@3.125.41.243 '
                                set -euo pipefail
                
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin docker.io
                                
                                docker pull $registry/$reponame/$appname:$BUILD_NUMBER

                                docker stop $appname || true
                                docker rm $appname || true

                                docker run -p 80:80 -d --name $appname $registry/$reponame/$appname:$BUILD_NUMBER
                                docker ps
                            '
                        """
                    }
                }
            }
        } 
    }
}

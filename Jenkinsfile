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
                steps {
                    withCredentials([
                        usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS'),
                        sshUserPrivateKey(credentialsId: 'ec2-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')
                    ]) {
                        sh """
                            ssh -i "\$SSH_KEY" -o StrictHostKeyChecking=no \$SSH_USER@54.93.200.200 "
                                set -euo pipefail
                
                                echo '$DOCKER_PASS' | docker login -u '$DOCKER_USER' --password-stdin docker.io
                                
                                docker pull $registry/$reponame/$appname:$BUILD_NUMBER

                                docker stop $appname || true
                                docker rm $appname || true

                                docker run -p 80:80 -d --name $appname $registry/$reponame/$appname:$BUILD_NUMBER
                                
                                sleep 5
                                
                                docker ps | grep $appname
                                
                                curl -f http://localhost || exit 1
                                echo 'deployed successfully'
                            "
                        """
                    }
                }
            } 
        }
    }

pipeline {
    agent any

    parameters {
        choice(name: 'BACKEND_COUNT', choices: ['1', '2'], description: 'Number of backend containers')
    }

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                script {
                    sh '''
                    docker network create app-network || true
                    docker rm -f backend1 backend2 || true
                    '''

                    if (params.BACKEND_COUNT == '1') {
                        sh '''
                        docker run -d --name backend1 --network app-network backend-app
                        '''
                    } else {
                        sh '''
                        docker run -d --name backend1 --network app-network backend-app
                        docker run -d --name backend2 --network app-network backend-app
                        '''
                    }
                }
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                script {
                    sh '''
                    docker rm -f nginx-lb || true
                    docker run -d --name nginx-lb --network app-network -p 80:80 nginx
                    '''

                    if (params.BACKEND_COUNT == '1') {
                        writeFile file: 'nginx.conf', text: '''
                        upstream backend {
                            server backend1:8080;
                        }

                        server {
                            listen 80;
                            location / {
                                proxy_pass http://backend;
                            }
                        }
                        '''
                    } else {
                        writeFile file: 'nginx.conf', text: '''
                        upstream backend {
                            server backend1:8080;
                            server backend2:8080;
                        }

                        server {
                            listen 80;
                            location / {
                                proxy_pass http://backend;
                            }
                        }
                        '''
                    }

                    sh '''
                    docker cp nginx.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    docker exec nginx-lb nginx -s reload
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully. Load balancing working!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}

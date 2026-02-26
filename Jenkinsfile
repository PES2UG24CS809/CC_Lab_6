pipeline {
    agent any

    parameters {
        string(name: 'BACKEND_COUNT', defaultValue: '2', description: 'Number of backend containers')
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
                    docker rm -f $(docker ps -aq --filter "name=backend") || true
                    '''

                    // Loop to create dynamic containers
                    for (int i = 1; i <= params.BACKEND_COUNT.toInteger(); i++) {
                        sh "docker run -d --name backend${i} --network app-network backend-app"
                    }
                }
            }
        }

        stage('Generate NGINX Config') {
            steps {
                script {
                    def config = "upstream backend {\n"

                    for (int i = 1; i <= params.BACKEND_COUNT.toInteger(); i++) {
                        config += "    server backend${i}:8080;\n"
                    }

                    config += "}\n\n"
                    config += "server {\n"
                    config += "    listen 80;\n"
                    config += "    location / {\n"
                    config += "        proxy_pass http://backend;\n"
                    config += "    }\n"
                    config += "}\n"

                    writeFile file: 'default.conf', text: config
                }
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                docker cp default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully. Load balancer is working.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}

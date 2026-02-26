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
                    int count = params.BACKEND_COUNT.toInteger()

                    sh 'docker network create app-network || true'

                    for (int i = 1; i <= count; i++) {
                        sh "docker rm -f backend${i} || true"
                        sh "docker run -d --name backend${i} --network app-network backend-app"
                    }
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
                  -p 80:80 nginx

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
}

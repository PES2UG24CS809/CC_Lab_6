stage('Deploy Backend Containers') {
    steps {
        script {
            sh 'docker network create app-network || true'
            sh 'docker rm -f $(docker ps -aq --filter "name=backend") || true'

            for (int i = 1; i <= params.BACKEND_COUNT.toInteger(); i++) {
                sh "docker run -d --name backend${i} --network app-network backend-app"
            }
        }
    }
}

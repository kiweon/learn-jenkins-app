pipeline {
    agent {
        docker {
                    image 'node:18-alpine'
                    reuseNode true
        }
    }

    stages {
        stage('Build') {
         
            steps {
                sh '''
                    ls -al
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -al
                '''
            }
        }
        stage('Test'){
            steps{
               echo 'Test stage'
               sh '''
                    test -f build/index.html
                    npm test
               '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}

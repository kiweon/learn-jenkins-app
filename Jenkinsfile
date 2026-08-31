pipeline {
    agent {
        docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
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
        stage('E2E'){
            steps{
                sh '''
				npm install -g serve
                serve -s build
                npx playwright test
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

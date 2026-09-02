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
				npm install serve
                node_modules/.bin/serve -s build & sleep 10
                npx playwright test --reporter=html
                '''
            }         
        }
        stage('Deploy'){
            steps{
                sh '''
                npm install netlify-cli@20.1.1
                npx netlify --version
                '''

            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}

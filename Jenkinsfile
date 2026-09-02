pipeline {
    agent {
        docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
        }
    }

    environment {
        NETLIFY_SITE_ID = 'f89b61dc-693d-4fcf-9264-0f1749a9c313'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
         
            steps {
                sh '''
                    echo '트리커 테스트 중 ...'
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
                echo "프로젝트 배포중... 사이트 아이디: $NETLIFY_SITE_ID"
                npx netlify status
                npx netlify deploy --dir=build --prod
                '''

            }
        }

        stage('Prod E2E'){
            environment {
                CI_ENVIRONMENT_URL ='https://elaborate-tanuki-fd5f22.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
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

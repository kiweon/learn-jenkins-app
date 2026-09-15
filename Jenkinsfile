pipeline {
   
    // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지

    
    agent none

    
    environment {
        NETLIFY_SITE_ID = 'f89b61dc-693d-4fcf-9264-0f1749a9c313'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    args "--entrypoint=''" 
                }
            }

            steps {

                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    echo "Hello S3!" > index.html
                    aws s3 cp index.html s3://skw-learn-jenkins/index.html
                '''
                }               
            }

        }
        stage('Build') {

            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }

            steps {
                sh '''
                    echo '트리 테스트 중 ...'
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

            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }         

            steps{
               echo 'Test stage'
               sh '''
                    test -f build/index.html
                    npm test
               '''
            }
        }
        stage('E2E'){
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }  

            steps{
                sh '''
				npm install serve
                node_modules/.bin/serve -s build & sleep 10
                npx playwright test --reporter=html
                '''
            }         
        }

        stage('Deploy staging'){

            agent {
                docker { image 'node:18-bullseye' } 
            }
            steps{
                sh '''
                npm install netlify-cli@20.1.1
                npx netlify --version
                echo "프로젝트 스테이징 배포중... 사이트 아이디: $NETLIFY_SITE_ID"
                npx netlify status
                npx netlify deploy --dir=build
                '''

            }
        }

        stage('Approval'){
            agent none
            steps {
                timeout(1) {
                 input message: '운영환경에 배포할까요?', ok: '네 배포합니다.'
                }
                
            }
        }

        stage('Deploy prod'){
            agent {
                docker { image 'node:18-bullseye' } 
            }
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
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }  

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
    
}

pipeline {
   
    // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지

    
    agent none

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME ='myjenkinsapp'
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        AWS_DOCKER_REGISTRY ='626036197706.dkr.ecr.ap-northeast-2.amazonaws.com'
        AWS_ECS_CLUSTER = 'busy-deer-618syt'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service_Prod'
        AWS_ECS_TD_PROD ='LearnJenkinsApp-TaskDefinition-Prod'
    }

   /* 
    environment {
        NETLIFY_SITE_ID = 'f89b61dc-693d-4fcf-9264-0f1749a9c313'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    */

    stages {

        
        
        stage('Build') {

            agent {
                docker {
                     image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                     reuseNode true
                     }
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

        stage('Build Docker image'){
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    args "-u root --entrypoint='' -v /var/run/docker.sock:/var/run/docker.sock"
                }
            }

           
            steps {

                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    yum install -y docker
                    docker build -t $APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                    docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }               
            }
            
        }

        stage('Deploy to AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    args "-u root --entrypoint=''" 
                }
            }

           
            steps {

                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    yum install jq -y
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }               
            }

        }

        

      /*
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
        */

    }
    
}

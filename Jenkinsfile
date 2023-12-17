pipeline {
    agent any
    environment {
        APP_NAME="phonebook"
        APP_REPO_NAME="bascher/${APP_NAME}-app"
        AWS_ACCOUNT_ID=sh(script:'aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        PATH="$PATH:/tmp:/var/lib/jenkins/workspace/phonebook-jenkins"
        }
    
        stages {
            stage('Create ECR Repo') {
                steps {
                    echo "Creating ECR Repo for ${APP_NAME}-app"
                    sh '''
                    aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                            aws ecr create-repository \
                            --repository-name ${APP_REPO_NAME} \
                            --image-scanning-configuration scanOnPush=true \
                            --image-tag-mutability MUTABLE \
                            --region ${AWS_REGION}
                    '''
                        }
                    }
            stage('Build App Docker Images for WEB') {
                steps {
                    echo 'Building App Images for web_server'
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    cd image_for_web_server
                    docker build -t clarusway-repo/phonebook-app:web .
                    docker tag clarusway-repo/phonebook-app:web ${ECR_REGISTRY}/${APP_REPO_NAME}:web
                    docker push ${ECR_REGISTRY}/${APP_REPO_NAME}:web
                    docker image ls
                    '''
                }
            }
            stage('Build App Docker Images for RESULT') {
                steps {
                    echo 'Building App Images for result_server'
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    cd image_for_result_server
                    docker build -t clarusway-repo/phonebook-app:result .
                    docker tag clarusway-repo/phonebook-app:result ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker push ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker image ls
                    '''
                }
            }


        }
}
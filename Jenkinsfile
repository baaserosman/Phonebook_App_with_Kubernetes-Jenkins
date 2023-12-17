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
                    cd 02_image_files/image_for_web_server
                    docker build -t my_repo/phonebook-app:web .
                    docker tag my_repo/phonebook-app:web ${ECR_REGISTRY}/${APP_REPO_NAME}:web
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
                    cd 02_image_files/image_for_result_server
                    docker build -t my_repo/phonebook-app:result .
                    docker tag my_repo/phonebook-app:result ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker push ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker image ls
                    '''
                }
            }
            stage('Installing eksctl, kubectl and creating EKS Cluster and deploying Phonebook Application') {
                steps {
                    echo 'Installing eksctl, kubectl and creating EKS Cluster and deploying Phonebook Application'
                    sh '''
                    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
                    eksctl version

                    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    kubectl version --short --client

                    eksctl create cluster -f cluster.yaml
                    
                    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml

                    sleep 200

                    kubectl apply -f ingress.yaml
                    kubectl apply -f mysql-configmap.yaml
                    kubectl apply -f mysql-deploy_possix.yaml
                    kubectl apply -f mysql-pv.yaml
                    kubectl apply -f mysql-pvc.yaml
                    kubectl apply -f mysql-secret-possix.yaml
                    kubectl apply -f mysql-svc.yaml
                    kubectl apply -f resultserver-deploy.yaml
                    kubectl apply -f resultserver-service.yaml
                    kubectl apply -f webserver-deploy.yaml
                    kubectl apply -f webserver-service.yaml
                    kubectl apply -f server-configmap.yaml

                    '''
                }


        }
    }
}
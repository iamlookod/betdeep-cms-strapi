pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    }
    stages {
        stage('Build') { 
            steps { 
                sh 'printenv'
                script{
                    app = docker.build("betdeep-cms-strapi")
                }
            }
        }
        stage('Test'){
            steps {
                echo 'Empty'
            }
        }
        stage('Push') {
            steps {
                sh "aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 701128946362.dkr.ecr.ap-southeast-1.amazonaws.com"
                sh "docker tag betdeep-cms-strapi:latest 701128946362.dkr.ecr.ap-southeast-1.amazonaws.com/betdeep-cms-strapi:latest"
                sh "docker tag betdeep-cms-strapi:latest 701128946362.dkr.ecr.ap-southeast-1.amazonaws.com/betdeep-cms-strapi:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                sh "docker push 701128946362.dkr.ecr.ap-southeast-1.amazonaws.com/betdeep-cms-strapi:latest"
                sh "docker push 701128946362.dkr.ecr.ap-southeast-1.amazonaws.com/betdeep-cms-strapi:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                echo ""
            }
        }
        stage('Deploy') {
            steps {
                script{
                    if (env.BRANCH_NAME == 'main') {
                        sh "helm -n betdeep-production upgrade -i betdeep-cms-strapi helm/ -f helm/values.yaml -f helm/values-production.yaml --atomic --set image.tag=${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    } else {
                        sh "helm -n betdeep-develop upgrade -i betdeep-cms-strapi helm/ -f helm/values.yaml -f helm/values-develop.yaml --atomic --set image.tag=${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    }
                } 
            }
        }
    }
}
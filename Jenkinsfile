pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('secret_key')
    }
    stages {
        stage('Build the new Project') {
            steps {
                git 'https://github.com/MANOJANASWARA/star-agile-banking-finance.git'
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t anasw/banking:v1 .'
                    sh 'docker images'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push anasw/banking:v1'
                }
            }
        }
        stage('Terraform Operations for Test Workspace') {
            steps {
                script {
                    sh '''
                    terraform workspace select test || terraform workspace new test
                    terraform init
                    terraform plan
                    terraform destroy -auto-approve
                    '''
                }
            }
        }
        stage('Terraform Apply for Test Workspace') {
            steps {
                script {
                    sh 'terraform apply -auto-approve'
                }
            }
        }
        stage('Terraform Operations for Production Workspace') {
            when {
                expression { return currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    sh '''
                    terraform workspace select prod || terraform workspace new prod
                    terraform init
                    if terraform state show aws_key_pair.example 2>/dev/null; then
                        echo "Key pair already exists in the prod workspace"
                    else
                        terraform import aws_key_pair.example key02 || echo "Key pair already imported"
                    fi
                    terraform destroy -auto-approve
                    terraform apply -auto-approve
                    '''
                }
            }
        }
    }
}

pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select the target environment'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = 'test'
        AWS_SECRET_ACCESS_KEY = 'test'
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Switch Workspace') {
            steps {
                sh "terraform workspace select ${params.ENVIRONMENT} || terraform workspace new ${params.ENVIRONMENT}"
            }
        }

        stage('Terraform Plan') {
            steps {
                sh "terraform plan -var-file=environments/${params.ENVIRONMENT}.tfvars -out=tfplan"
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: "Apply changes for [${params.ENVIRONMENT}]?", ok: "Approve"
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sh "terraform apply tfplan"
            }
        }
    }

    post {
        success {
            mail to: 'sofiabahaa93@gmail.com',
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Pipeline succeeded for environment: ${params.ENVIRONMENT}\n${env.BUILD_URL}"
        }
        failure {
            mail to: 'sofiabahaa93@gmail.com',
                 subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Pipeline failed for environment: ${params.ENVIRONMENT}\nCheck logs: ${env.BUILD_URL}console"
        }
    }
}

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
                echo "Simulating terraform init (network step skipped for demo)..."
                sh 'terraform version'
            }
        }

        stage('Switch Workspace') {
            steps {
                echo "Simulating workspace switch to: ${params.ENVIRONMENT}"
            }
        }

        stage('Terraform Plan') {
            steps {
                echo "Simulating terraform plan for ${params.ENVIRONMENT}..."
                echo "Plan: 3 to add, 0 to change, 0 to destroy. (simulated)"
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
                echo "Simulating terraform apply for ${params.ENVIRONMENT}..."
                echo "Apply complete! Resources: 3 added, 0 changed, 0 destroyed. (simulated)"
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

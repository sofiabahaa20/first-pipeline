pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select which environment to deploy'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        TF_VAR_FILE           = "environments/${params.ENVIRONMENT}.tfvars"
    }

    // Configure the matching GitHub webhook in the repo settings
    // (Settings > Webhooks > payload URL: http://<jenkins-url>/github-webhook/)
    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Execution') {
            stages {

                stage('Terraform Init') {
                    steps {
                        sh 'terraform init -input=false'
                    }
                }

                stage('Workspace Select') {
                    steps {
                        sh """
                            terraform workspace select ${params.ENVIRONMENT} || \
                            terraform workspace new ${params.ENVIRONMENT}
                        """
                    }
                }

                stage('Plan') {
                    steps {
                        sh "terraform plan -var-file=${TF_VAR_FILE} -out=tfplan -input=false"
                    }
                }

                stage('Approve') {
                    steps {
                        script {
                            input(
                                id: 'ApplyApproval',
                                message: "Approve Terraform apply for '${params.ENVIRONMENT}'?",
                                ok: 'Approve'
                            )
                        }
                    }
                }

                stage('Apply') {
                    steps {
                        sh 'terraform apply -input=false -auto-approve tfplan'
                    }
                }
            }
        }
    }

    post {
        success {
            mail to: 'sofiabahaa93@gmail.com',
                 subject: "SUCCESS: Terraform Apply - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Terraform apply succeeded for environment: ${params.ENVIRONMENT}\n\nBuild: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'sofiabahaa93@gmail.com',
                 subject: "FAILURE: Terraform - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Terraform pipeline failed (or was aborted) for environment: ${params.ENVIRONMENT}\n\nCheck logs: ${env.BUILD_URL}console"
        }
        aborted {
            mail to: 'sofiabahaa93@gmail.com',
                 subject: "ABORTED: Terraform Apply - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The apply approval was rejected/timed out for environment: ${params.ENVIRONMENT}."
        }
    }
}

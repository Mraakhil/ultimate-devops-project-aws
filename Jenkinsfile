pipeline {
    agent any

    // Defines a parameter to choose between creating or destroying the EKS cluster
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose whether to apply or destroy the Terraform EKS module')
    }

    environment {
        // --- TARGET MODULE LOCATION ---
        // Tells Jenkins where main.tf and other terraform files live
        TF_DIR = 'eks-install'

        // AWS Jenkins Credentials ID and target region
        AWS_CREDENTIALS_ID = 'your-aws-credentials-id'
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls repository code into workspace
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                // dir() switches context into the 'eks-install' directory where main.tf resides
                dir("${env.TF_DIR}") {
                    withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                // Runs terraform plan inside 'eks-install', automatically reading main.tf
                dir("${env.TF_DIR}") {
                    withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh "terraform plan ${params.ACTION == 'destroy' ? '-destroy' : ''} -out=tfplan"
                    }
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Review plan for folder '${env.TF_DIR}'. Proceed with ${params.ACTION}?", ok: 'Proceed'
            }
        }

        stage('Terraform Execute') {
            steps {
                dir("${env.TF_DIR}") {
                    withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        script {
                            if (params.ACTION == 'apply') {
                                sh 'terraform apply -auto-approve tfplan'
                            } else if (params.ACTION == 'destroy') {
                                sh 'terraform apply -destroy -auto-approve'
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs() // Cleans up workspace after execution
        }
    }
}
pipeline {
    agent any
    
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose whether to apply or destroy the Terraform EKS module')
    }

    environment {
        TF_DIR = 'eks-install'
        AWS_CREDENTIALS_ID = 'your-aws-credentials-id'
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir("${env.TF_DIR}") {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir("${env.TF_DIR}") {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
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
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}", accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
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
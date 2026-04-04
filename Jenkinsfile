pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Auto approve apply/destroy?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select action')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'ap-south-1'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Shafi23/terraform-jenkins-pipeline.git'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Plan') {
            steps {
                script {
                    if (params.action == 'apply') {
                        sh 'terraform plan -out=tfplan'
                        sh 'terraform show -no-color tfplan > tfplan.txt'
                    } else {
                        sh 'terraform plan -destroy -out=destroy.tfplan'
                        sh 'terraform show -no-color destroy.tfplan > destroy.txt'
                    }
                }
            }
        }

        stage('Apply / Destroy') {
            steps {
                script {

                    if (params.action == 'apply') {

                        if (!params.autoApprove) {
                            def plan = readFile 'tfplan.txt'
                            input message: "Approve Terraform Apply?",
                                  parameters: [text(name: 'Plan', defaultValue: plan)]
                        }

                        // Avoid stale plan issue (best practice)
                        sh 'terraform apply -auto-approve'

                    } else if (params.action == 'destroy') {

                        if (!params.autoApprove) {
                            def plan = readFile 'destroy.txt'
                            input message: "Approve Terraform Destroy?",
                                  parameters: [text(name: 'Destroy Plan', defaultValue: plan)]
                        }

                        sh 'terraform destroy -auto-approve'
                    }
                }
            }
        }
    }
}
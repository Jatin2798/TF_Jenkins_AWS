pipeline {
    agent any
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        PATH = "${env.PATH};C:\\Terraform" // Add Terraform to the PATH
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    dir('terraform') {
                        git url: 'https://github.com/Jatin2798/TF_Jenkins_AWS.git', branch: 'main'
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                script {
                    // Run Terraform init
                    def initStatus = bat returnStatus: true, script: """
                        cd terraform
                        terraform init -no-color > terraform_init.log 2>&1
                    """
                    archiveArtifacts artifacts: 'terraform/terraform_init.log', allowEmptyArchive: true
                    if (initStatus != 0) {
                        error "Terraform init failed. Check terraform_init.log for details."
                    }

                    // Run Terraform plan
                    def planStatus = bat returnStatus: true, script: """
                        cd terraform
                        terraform plan -out=tfplan -no-color > terraform_plan.log 2>&1
                    """
                    archiveArtifacts artifacts: 'terraform/terraform_plan.log', allowEmptyArchive: true
                    if (planStatus != 0) {
                        error "Terraform plan failed. Check terraform_plan.log for details."
                    }

                    // Generate human-readable plan
                    bat """
                        cd terraform
                        terraform show -no-color tfplan > tfplan.txt
                    """
                    archiveArtifacts artifacts: 'terraform/tfplan.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }
            steps {
                script {
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Apply') {
            steps {
                script {
                    // Apply Terraform plan
                    def applyStatus = bat returnStatus: true, script: """
                        cd terraform
                        terraform apply -input=false tfplan -no-color > terraform_apply.log 2>&1
                    """
                    archiveArtifacts artifacts: 'terraform/terraform_apply.log', allowEmptyArchive: true
                    if (applyStatus != 0) {
                        error "Terraform apply failed. Check terraform_apply.log for details."
                    }
                }
            }
        }
    }
}

pipeline {
    agent any  // Use any agent to run the pipeline
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')  // Ensure correct credential ID
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')  // Ensure correct credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Check out the repository into the 'terraform' directory
                    dir('terraform') {
                        git url: 'https://github.com/Jatin2798/TF_Jenkins_AWS.git'
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                // Initialize Terraform and generate plan
                sh 'pwd; cd terraform/; terraform init'
                sh 'pwd; cd terraform/; terraform plan -out=tfplan'
                sh 'pwd; cd terraform/; terraform show -no-color tfplan > tfplan.txt'
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
                    // Read the generated plan and ask for user input to approve
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Apply') {
            steps {
                // Apply the Terraform plan
                sh 'pwd; cd terraform/; terraform apply -input=false tfplan'
            }
        }
    }
}

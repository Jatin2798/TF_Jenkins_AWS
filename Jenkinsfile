pipeline {
    agent any
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Change the branch name from 'master' to 'main'
                    dir('terraform') {
                        git url: 'https://github.com/Jatin2798/TF_Jenkins_AWS.git', branch: 'main'
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                bat 'echo %CD%; cd terraform/; terraform init'
                bat 'echo %CD%; cd terraform/; terraform plan -out=tfplan'
                bat 'echo %CD%; cd terraform/; terraform show -no-color tfplan > tfplan.txt'
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
                bat 'echo %CD%; cd terraform/; terraform apply -input=false tfplan'
            }
        }
    }
}

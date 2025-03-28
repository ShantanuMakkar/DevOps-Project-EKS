properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )])
])

pipeline {
    agent any
    stages {

        stage('Preparing') {
            steps {
                sh 'echo preparing..'
            }
        }

        stage('Git Pulling') {
            steps {
                sh 'echo preparing to pull the git repo..'
                git branch: 'master', url: 'https://github.com/ShantanuMakkar/DevOps-Project-EKS.git'
            }
        }

        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'echo connecting to aws account..'
                sh 'echo testing the s3 buckets for aws connectivity..'
                sh 'aws s3 ls'
                sh 'echo aws is connected..'
                sh 'echo starting terraform initialization..'    
                sh 'terraform -chdir=eks/ init'
                }
            }
        }

        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'echo starting terraform validation'    
                sh 'terraform -chdir=eks/ validate'
                }
            }
        }

        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {    
                        if (params.Terraform_Action == 'plan') {
                            sh 'echo starting terraform plan..'
                            sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                        }   
                        else if (params.Terraform_Action == 'apply') {
                            sh 'echo starting terraform apply..'
                            sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                        }   
                        else if (params.Terraform_Action == 'destroy') {
                            sh 'echo starting terraform destroy..'
                            sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                        } 
                        else {
                            error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
}

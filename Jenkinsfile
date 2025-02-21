pipeline {
    agent any
    environment {
        GITNAME = 'hansususu'
        GITEMAIL = 'hansubin0039@gmail.com'
        GITWEBADD = 'https://github.com/hansususu/bom-test.git'
        GITSSHADD = 'git@github.com:hansususu/argo-test.git'

        AWS_CREDENTIAL_NAME = 'aws_cre'
        ECR_PATH = '575108933149.dkr.ecr.ap-northeast-1.amazonaws.com'
        IMAGE_NAME = '575108933149.dkr.ecr.ap-northeast-1.amazonaws.com/bombi/brigde'
        REGION = 'ap-northeast-2'

        GITCREDENTIAL = 'git_cre'
    }
    stages {
        stage('Checkout Github') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])
            }
            post {
                failure {
                    sh "echo clone failed"
                }
                success {
                    sh "echo clone success"
                }
            }
        }
        stage('Docker Image Push') {
            steps {
                script {
                    // Authenticate with ECR using AWS CLI
                    sh 'aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_PATH}'
                    
                }
            }
        }

        stage('EKS manifest file update') {
            steps {
                git credentialsId: GITCREDENTIAL, url: GITSSHADD, branch: 'main'
                sh "git config --global user.email ${GITEMAIL}"
                sh "git config --global user.name ${GITNAME}"
                sh "sed -i 's@${IMAGE_NAME}:.*@${IMAGE_NAME}:${currentBuild.number}@g' deployment/bridge-deployment.yaml"

                sh "git add ."
                sh "git branch -M main"
                sh "git commit -m 'fixed tag ${currentBuild.number}'"
                sh "git remote remove origin"
                sh "git remote add origin ${GITSSHADD}"
                sh "git push origin main"
            }
            post {
                failure {
                    sh "echo manifest update failed"
                }
                success {
                    sh "echo manifest update success"
                }
            }
        }
       
    }
}
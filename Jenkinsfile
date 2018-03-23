#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        NODE_VERSION='4.8.7'   
        NVM_VERSION='0.33.8'
    }
    tools {
        nodejs "node-${env.NODE_VERSION}"
    }
    stages{
        stage('Checkout') {
            steps{
                checkout scm
            }
        }
         stage('Configure Jenkins'){
             steps {
                sh '''
                    ls jenkins-tools/packer >>/dev/null 2>&1 && exit 0
                    echo 'Downloading Packer'
                    mkdir -p jenkins-tools
                    curl -o jenkins-tools/packer.zip 'https://releases.hashicorp.com/packer/1.2.1/packer_1.2.1_linux_amd64.zip'
                    unzip jenkins-tools/packer.zip && mv packer jenkins-tools/
                    chmod +x jenkins-tools/packer 
                    rm jenkins-tools/packer.zip
                    echo 'Successfully downloaded Packer'
                '''
             }
         }
        stage('Install dependencies/ build'){
            steps{
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps{
                sh 'npm pack >node-pack-name'
                withCredentials([file(credentialsId: 'limitd-packer-secrets.json', variable: 'PACKER_SECRETS')]){
                    sh '''jenkins-tools/packer build \
                    -var "ami_name=$JOB_NAME-$(cat node-pack-name | cut -d'.' -f1)"\
                    -var "node_version='$NODE_VERSION'"\
                    -var "nvm_version='$NVM_VERSION'"\
                    -var "node_pack=$(cat node-pack-name)"\
                    -var-file=${PACKER_SECRETS}\
                    deploy/packer.json
                '''
                }
            }
        }
        stage('Cleanup') {
            steps{
                sh 'rm -rf node_modules/'
            }
        }
    }
}
pipeline {
    agent any
    tools {
        nodejs "node-4.8.7"
    }
    stages{
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
        stage('Checkout') {
            steps{
                checkout scm
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
                sh 'npm pack'
                withCredentials([file(credentialsId: 'limitd-packer-secrets.json', variable: 'PACKER_SECRETS')]){
                    sh '''jenkins-tools/packer build \
                    -var ami_name='mcordischi-limitd'\
                    -var node_version='4.8.7'\
                    -var node_pack='limitd-5.5.2.tgz'\
                    -var-file=${PACKER_SECRETS}\
                    build/packer.json
                '''
                }
            }
        }
        stage('Cleanup') {
            steps{
                echo 'Cleanup'
                //sh 'rm -rf node_modules/'
            }
        }
    }
}
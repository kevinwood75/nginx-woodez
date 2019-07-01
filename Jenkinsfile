pipeline {
    agent none
    stages {
        stage('Send alert that build has started'){
            agent { label 'appserver'}
            steps {
                notifyStarted()
            }
        }
        
        stage('Git Checkout'){
            agent { label 'appserver'}
            steps {
                git credentialsId: 'kevinwood75', url: 'https://github.com/kevinwood75/nginx-woodez.git', branch: 'master'
            }
        }
        stage('Build Docker Image'){
            agent { label 'appserver' }
            steps {
                sh 'docker build -t kwood475/nginx-woodez:2.0.0 .'
            }
        }
        stage('Push Docker Image'){
            agent { label 'appserver'}
            steps {
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerhub')]) {
                   sh "docker login -u kwood475 -p ${dockerhub}"
                }
                sh 'docker push kwood475/nginx-woodez:2.0.0'
            }
        }
        
        stage('get appproval for prod release'){
            agent { label 'rasp' }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    script {
                      def INPUT_PARAMS = input id: 'Release', message: 'Approve release to prod', ok: 'Approve', parameters: [booleanParam(defaultValue: true, description: 'approver status', name: 'approver_status')], submitter: 'kwood', submitterParameter: 'approvername'
                    }
                }  
            }
        }
        
        stage('Remove older container before release'){
            agent { label 'rasp' }
            steps {
                sh 'docker rm -f nginx-woodez || true'
            }
        }

        stage('Release Container prod'){
            agent { label 'rasp' }
            steps {
                sh 'docker run --name nginx-woodez -p 80:80 -d kwood475/nginx-woodez:2.0.0'
            }
        } 
    }
}

def notifyStarted() {
  slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
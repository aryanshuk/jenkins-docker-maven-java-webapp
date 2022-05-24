pipeline {
    agent {
        label "maven-build-node"
    }
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/aryanshuk/jenkins-docker-maven-java-webapp.git'
            }
        }
        
        stage('Build By Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Own Docker Image') {
            steps {
                sh "sudo docker build -t aryanshuk06/java-webapp:${BUILD_TAG} ."
            }
        }
        
        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_PASSWD', variable: 'DOCKER_HUB_PASSWD')]) {
                    sh "sudo docker login -u aryanshuk06 -p ${DOCKER_HUB_PASSWD}"
                }
                sh 'sudo docker push aryanshuk06/java-webapp:${BUILD_TAG}'
            }
        }
        stage('Deploy WebApp in DEV Env') {
            steps {
                sh 'sudo docker rm -f my-java-webapp'
                sh 'sudo docker run -d -p 8080:8080 --name my-java-webapp aryanshuk06/java-webapp:${BUILD_TAG}'
            }
        }
        
        stage('Deploy WebApp in QA Env') {
            steps {
                sshagent(['QA_ENV_SSH_CRED']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.235.16.118 sudo docker rm -f my-java-webapp'
                    sh 'ssh ec2-user@13.235.16.118 sudo docker run -d -p 8080:8080 --name my-java-webapp aryanshuk06/java-webapp:${BUILD_TAG}'
                }
            }
        }
        stage('QAT Test') {
            steps {
                sh 'sleep 10'
                sh 'curl --silent http://13.235.16.118:8080/java-web-app/ | grep India ' 
            }
        }
        stage('QAT Approval') {
            steps {
                input(message: "Release to Production Environment?")
            }
        }
        stage('Deploy WebApp in Prod Env') {
            steps {
                sshagent(['PROD_ENV_CRED']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.232.234.158 sudo docker rm -f my-java-webapp'
                    sh 'ssh ec2-user@13.232.234.158 sudo docker run -d -p 8080:8080 --name my-java-webapp aryanshuk06/java-webapp:${BUILD_TAG}'
                }
            }
        }
    }
}

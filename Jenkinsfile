pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO='ksahadeva/node-app'
        IMAGE_TAG="${params.IMAGE_TAG}"
        DEV_USER='ubuntu'
        DEV_URL='52.66.205.49'
        CONTAINER_NAME="${params.CONTAINER_NAME}"
        OUT_PORT="${params.OUT_PORT}"
        BRANCH='${params.BRANCH}'
        
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: '$BRANCH', url: 'https://github.com/9148123118/nodejs-docker-project.git'
            }
        }
        stage('Build docker image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_REPO:$IMAGE_TAG .'
            }
        }
        stage('login and push image to dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER-HUB-CREDS', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh 'docker login -u ${USERNAME} -p ${PASSWORD} docker.io'
                    sh 'docker push $DOCKER_HUB_REPO:$IMAGE_TAG'
              }
                
            }
        }
        stage('Stop & remove container') {
            steps {
                sshagent(['dev-creds']) {
                    sh "ssh -o StrictHostKeyChecking=no $DEV_USER@$DEV_URL sudo docker stop $CONTAINER_NAME"
                    sh 'ssh $DEV_USER@$DEV_URL sudo docker rm -f $CONTAINER_NAME'
                    
                    
              }
            }
        }
        stage('Deploy container') {
            steps {
                sshagent(['dev-creds']) {
                    sh 'ssh -o StrictHostKeyChecking=no $DEV_USER@$DEV_URL sudo docker pull $DOCKER_HUB_REPO:$IMAGE_TAG' 
                    sh 'ssh $DEV_USER@$DEV_URL sudo docker run --name $CONTAINER_NAME -d -p $OUT_PORT:8080 $DOCKER_HUB_REPO:$IMAGE_TAG'
            
              }
            }
        }
    }
}

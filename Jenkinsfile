pipeline {
    // target node yang akan menjalnkan ci/cd or code
    agent {
           label 'master02'
    }
    stages{
//        stage("checkout"){
//            steps{
//                checkout scm
//            }
//        }
        // create image from Dockerfile github
        stage("Build Image"){
            steps{
                sh 'sudo docker build -t registry-nexus.cloud/apps-caculator:latest .'
            }
        }
        // push image to repository local, sure node can hit domain repository local
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'registry-docker', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'sudo docker login registry-nexus.cloud -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh 'sudo docker push registry-nexus.cloud/apps-caculator:latest'
                    sh 'sudo docker logout'
                }
            }
        }
        // running container
        stage('Docker RUN') {
            steps {
                sh 'sudo docker run -d -p 3000 --name app-caculator  registry-nexus.cloud/apps-caculator:latest'
            }
        }
        stage('Remove Container & create deployment') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'registry-docker', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                sh 'docker rm -f app-caculator'
                sh 'kubectl apply -f caculator.yaml'
                sh 'kubectl create secret docker-registry admin-repo --docker-server=registry-nexus.cloud --docker-username=$DOCKER_USERNAME --docker-password=$DOCKER_PASSWORD --namespace=apps-caculator'
                }
            }
        }
    }
}


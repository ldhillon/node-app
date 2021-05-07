pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t ldhillon/nodeapp:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                    sh "docker login -u ldhillon -p ${dockerHubPwd}"
                    sh "docker push ldhillon/nodeapp:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy App to k8s') {
             steps {
                 sh "chmod +x changeTag.sh"
                 sh "./changeTag.sh ${DOCKER_TAG}"
                 sshagent(['docker-jenkins-pvt-key']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ldhillon@mastertools.lsd.com:/home/ldhillon/jenkinsStage/"
                     script{
                         try{
                            sh "ssh ldhillon@mastertools.lsd.com kubectl apply -f ./jenkinsStage/node-app-pod.yml -f ./jenkinsStage/services.yml"
                         }catch(error){
                             sh "ssh ldhillon@mastertools.lsd.com kubectl create -f ./jenkinsStage/node-app-pod.yml -f ./jenkinsStage/services.yml"
                         }
                     }
                 }
             }
        }
     }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}

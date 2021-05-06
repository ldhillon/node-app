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
        stage('Deploy App') {
             steps {
                 sh "chmod +x changeTag.sh"
                 sh "./changeTag.sh ${DOCKER_TAG}"
                 script {
                     kubernetesDeploy(configs: "node-app-pod.yml, services.yml", kubeconfigId: "mykubeconfig")
                 }
             }
        }
     }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}

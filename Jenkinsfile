pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t scmmohammad/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'scmmohammad', variable: 'dockerHubPwd')]) {
                    sh "docker login -u scmmohammad -p ${dockerHubPwd}"
                    sh "docker push scmmohammad/nodeapp:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@54.196.20.7:/home/ubuntu/"
                    script{
                        try{
                            sh "ubuntu@54.196.20.7 kubectl apply -f ."
                        }catch(error){
                            sh "ubuntu@54.196.20.7 kubectl create -f ."
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

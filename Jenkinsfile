pipeline {
    agent 'any'
    stages {
        stage ('Pull Code') {
            steps {
                sh 'echo PULLING CODE FROM GITHUB'
                git 'https://github.com/tlprgit/task.git'
            }
        }
        stage ('Building docker image from dockerfile and run containers using docker-compose') {
            steps {
                sh 'docker kill $(docker ps -q)'
                sh 'docker rm $(docker ps -a -q)'
                sh 'docker rmi $(docker images -q) -f'
                sh 'docker build -t task:latest .'
                sh 'docker tag task:latest tlprdocker/task:latest'
                sh 'docker push tlprdocker/task:latest'
                sh 'docker-compose up -d'
            }
        }
        stage('Task:2 Scanning Docker Images using TRIVY') {
            steps {
                sh 'trivy image task:latest'
                sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL testjob:latest'
            }
        }
        stage('Docker image and Deploy into k8s') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }
}

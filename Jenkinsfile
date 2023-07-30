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
                sh 'docker stop $(docker ps -a -q)'
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
                sh 'trivy image --reset'
                sh 'trivy image tlprdocker/task:latest'
                sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL tlprdocker/task:latest'
            }
        }
        stage ('Sonarqube') {
            script {
                    def scannerHome = tool name: 'sonarscanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation';
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
            }
        }
        
    }
}

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
                sh 'docker build -t task:latest .'
                sh 'docker tag task:latest tlprdocker/task:latest'
                sh 'docker push tlprdocker/task:latest'
                sh 'docker-compose up -d'
            }
        }
            stage('SonarQube Analysis') {
                steps{
                    script {
                def scannerHome = tool 'sonar'
                withSonarQubeEnv('SonarQube') {
                    sh """/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube/bin/sonar-scanner \
                    -D sonar.projectVersion=1.0-SNAPSHOT \
                    -D sonar.login=admin \
                    -D sonar.password=tlpr \
                    -D sonar.projectBaseDir=/var/lib/jenkins/workspace/test1_master \
                    -D sonar.projectKey=sonar \
                    -D sonar.sourceEncoding=UTF-8 \
                    -D sonar.language=js \
                    -D sonar.sources=config,controllers,models,test,views \
                    -D sonar.tests=test1/test \
                    -D sonar.host.url=http://3.84.141.102:9000/"""
                }
                    }
                }
            }
        stage('Task:2 Scanning Docker Images using TRIVY') {
            steps {
                sh 'trivy image testjob:latest'
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

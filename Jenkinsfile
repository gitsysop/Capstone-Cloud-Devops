pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e *.html'
                sh 'hadolint Dockerfile'
            }
        }
        stage('Build Docker Image') {
            steps {
                  sh 'docker build -t capstone-cloud-devops .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker tag capstone-cloud-devops dockersysop/capstone-cloud-devops"
                    sh 'docker push dockersysop/capstone-cloud-devops'
                }  
            }
        }
        stage('Deploying') {
            steps{
                withAWS(credentials: 'aws', region: 'us-west-2') {
                    sh "aws eks --region us-west-2 update-kubeconfig --name capstone"
                    sh "kubectl config use-context arn:aws:eks:us-west-2:570954939712:cluster/capstone"
                    sh "kubectl apply -f deployment.yml"
                    sh "kubectl set image deployments/capstone-cloud-devops capstone-cloud-devops=dockersysop/capstone-cloud-devops:latest"
                    sh 'kubectl rollout status deployment capstone-cloud-devops'
                    sh 'kubectl get deployment capstone-cloud-devops'
                }
            }
        }
        stage("Cleaning up") {
            steps{
                sh "docker system prune"
            }
        }
    }
}

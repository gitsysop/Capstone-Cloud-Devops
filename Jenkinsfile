pipeline {
    agent any
    stages {

        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e *.html'
            }
        }
        stage('Build Docker Image') {
            steps {
                  sh 'docker build -t capstone-cloud-devops .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([url: "", credentialsId: "dockerhub"]) {
                    sh "docker tag capstone-cloud-devops dockersysop/capstone-cloud-devops"
                    sh 'docker push dockersysop/capstone-cloud-devops'
                }  
            }
        }
        stage('Deploying') {
            steps{
                echo 'Deploying to AWS...'
                withAWS(credentials: 'aws', region: 'us-west-2') {
                    sh "aws eks --region us-west-2 update-kubeconfig --name capstone"
                    sh "kubectl config use-context arn:aws:eks:us-west-2:988212813982:cluster/capstone"
                    sh "kubectl set image deployments/capstone-cloud-devops capstone-cloud-devops=dockersysop/capstone-cloud-devops:latest"
                    sh "kubectl apply -f deployment.yml"
                    /*sh "kubectl get nodes"
                    sh "kubectl get deployment"
                    sh "kubectl get pod -o wide"
                    sh "kubectl get service/capstone-cloud-devops"*/
                }
            }
        }
        stage("Cleaning up") {
            steps{
                echo 'Cleaning up...'
                sh "docker system prune"
            }
        }
    }
}

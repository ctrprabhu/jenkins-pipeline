pipeline {
    agent any
    
    stages{
        stage('SonarQube Analysis') {
      steps {
        script {
          // requires SonarQube Scanner 2.8+
          scannerHome = tool 'SonarScanner'
        }
        withSonarQubeEnv('SonarQube Server') {
          sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=sqa_be0c5cb6c696fb64f99de1f65929c86b321da4e2"
        }
      }
    }

        stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t ctrprabhu/python-http-server .'
            }
        }
    }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name python-app ctrprabhu/python-http-server && sleep 10 && docker stop python-app'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubpass')]) {
                    sh 'docker login -u ctrprabhu --password ${DockerHubpass}' }
                    sh 'docker push ctrprabhu/python-http-server'
                }
            }
        }    
}
        post {
        always {
            // Always executed
                sh 'docker rm python-app'
        }
        success {
            // on sucessful execution
            sh 'docker logout'   
        }
    }
      stage('Deploying App to Kubernetes') {
          steps {
              script {
                   kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
        }
      }
    }
}

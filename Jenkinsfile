pipeline{
    agent any
    // tools{
    //     jdk 'jdk17'
    //     nodejs 'node16'
    // }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/rameshkumarvermagithub/Valentine-Day-DevOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=valentine-day-devOps-project \
                    -Dsonar.projectKey=valentine-day-devOps-project'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/valentine-day-devops-project ."
                       // sh "docker tag valentine-day-devops-project rameshkumarverma/valentine-day-devops-project:latest"
                       sh "docker push rameshkumarverma/valentine-day-devops-project:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/valentine-day-devops-project:latest > trivyimage.txt"
            }
        }
        stage("deploy_docker"){
            steps{
                sh "docker stop valentine-day-devops-project || true"  // Stop the container if it's running, ignore errors
                sh "docker rm valentine-day-devops-project || true" 
                sh "docker run -d --name valentine-day-devops-project -p 80:80 rameshkumarverma/valentine-day-devops-project"
            }
        }
      // stage('Deploy to Kubernetes') {
      //       steps {
      //           script {
      //               // dir('K8S') {
      //                   withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
      //                       // Apply deployment and service YAML files
      //                       sh 'kubectl apply -f deployment.yml'
      //                       // sh 'kubectl apply -f service.yml'

      //                       // Get the external IP or hostname of the service
      //                       // def externalIP = sh(script: 'kubectl get svc amazon-service -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"', returnStdout: true).trim()

      //                       // Print the URL in the Jenkins build log
      //                       // echo "Service URL: http://${externalIP}/"
      //                   }
      //               // }
      //           }
      //       }
      //   }

    }
}

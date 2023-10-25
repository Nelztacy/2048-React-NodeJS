pipeline{
    agent any
    tools{
        jdk 'java17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from GitHub'){
            steps{
                git branch: 'main', url: 'https://github.com/Nelztacy/2048-React-NodeJS.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DPC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t nelzone/2048:latest .'
                }
            }
        }
        stage('Push image to DockerHub'){
            steps {
                script {
                  def dockerImage = docker.build("2048:latest", ".")
                  docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                      sh '''
                      withCredentials([string(credentialsId: 'dockerhub', variable: '')]) {
                      sudo docker tag 2048:latest nelzone/2048:latest
                      sudo docker push nelzone/2048:latest
                      '''
                  }
                }
            }
        }
        stage('Trivy Image Scan'){
            steps{
                sh "trivy image nelzone/2048:latest > trivy.txt" 
            }
        }
    }
}
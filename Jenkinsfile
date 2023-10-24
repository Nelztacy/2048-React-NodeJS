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
        stage('Docker Build $ Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker') {
                        sh '''
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 855956050827.dkr.ecr.us-east-1.amazonaws.com
                        sudo docker build -t 2048 .
                        sudo docker tag 2048:latest 855956050827.dkr.ecr.us-east-1.amazonaws.com/2048:latest
                        sudo docker push 855956050827.dkr.ecr.us-east-1.amazonaws.com/2048:latest '''
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
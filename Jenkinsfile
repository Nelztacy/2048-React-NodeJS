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
        stage('Docker Build & Push') {
        steps {
        script {
            def dockerImage = docker.build("2048:latest", ".")
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                sh '''
                echo "$DOCKER_HUB_CREDENTIALS_PSW" | sudo docker login -u $DOCKER_CREDENTIAL --password-stdin
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
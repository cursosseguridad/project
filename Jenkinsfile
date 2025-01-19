pipeline{
    agent any
    tools{
        jdk 'jdk17'
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
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/cursosseguridad/project'
            }
        }
        stage("Analisis Sonarqube"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -X -Dsonar.projectName=usachdevsecops \
                    -Dsonar.projectKey=jt '''
                }
            }
        }
        stage("Quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                }
            }
        }
        stage('Instalacion de Dependencias') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t jt ."
                       sh "docker tag jt dockerhubusach/docker:latest "
                       sh "docker push dockerhubusach/docker:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image dockerhubusach/docker:latest > trivy.txt"
            }
        }
        stage('Despliegue a container'){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker run -d --name devsecops -p 3002:3002 dockerhubusach/docker:latest'
                    }
                }
            }
        }

    }
}

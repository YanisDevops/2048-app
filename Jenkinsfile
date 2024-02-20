pipeline {
    agent any 
    tools {
        jdk 'jdk'
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/uniquesreedhar/2048-app.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=2048-game \
                        -Dsonar.projectKey=2048-game '''
                    }
                
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Installing Dependencies') {
            steps {
                sh 'npm install'
                
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                        sh 'docker system prune -f'
                        sh 'docker container prune -f'
                        sh 'docker build -t 2048-game .'
                    }
                    
                }
            }
        }
        stage("Docker Image Pushing") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                        sh 'docker tag 2048-game sreedhar8897/2048-game:${BUILD_NUMBER}'
                        sh 'docker push sreedhar8897/2048-game:${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image sreedhar8897/2048-game:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
    }
}

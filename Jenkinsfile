pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'master', url: 'https://github.com/chintu4494/2048_React_K8S.git'
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                        -Dsonar.projectKey=Game \
                        -Dsonar.login=squ_b3708f70d80b15eb45d2a2a63a14c4c5000b7fea"""
                }
            }
        }

        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-creds'
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t chintu4494:latest ."
                        sh "docker tag chintu4494:latest chintu4494:latest"
                        sh "docker push chintu4494:latest"
                    }
                }
            }
        }

        stage("TRIVY") {
            steps {
                sh "trivy image chintu4494:latest > trivy.txt"
            }
        }

        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name 2048 -p 3000:3000 chintu4494:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(
                        caCertificate: '',
                        clusterName: '',
                        contextName: '',
                        credentialsId: 'k8s',
                        namespace: '',
                        restrictKubeConfigAccess: false,
                        serverUrl: ''
                    ) {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}

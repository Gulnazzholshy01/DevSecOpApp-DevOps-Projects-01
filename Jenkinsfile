pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-pat', url: 'https://github.com/Gulnazzholshy01/DevSecOpProject-App.git'
            }
        }
        stage('Compilation') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame\
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-cred') {
                        sh "docker build -t gulnaz1357/boardgame:latest ."
                    }
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-fs-report.html gulnaz1357/boardgame:latest'
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-cred') {
                        sh "docker push gulnaz1357/boardgame:latest "
                    }
                }
            }
        }
        stage('Deploy to K8S') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.5.47:6443') {
                    sh "kubectl apply -f deployment-service.yml"
                }
            }
        }
        stage('Verify K8S deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.5.47:6443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
    post {
        always {
            // script {
            //     def jobName = env.JOB_NAME
            //     def buildNumber = env.BUILD_NUMBER
            //     def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            //     def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            //     def body = """
            //         <html>
            //         <body>
            //         <div style="border: 4px solid ${bannerColor}; padding: 10px;">
            //         <h2>${jobName} - Build ${buildNumber}</h2>
            //         <div style="background-color: ${bannerColor}; padding: 10px;">
            //         <h3 style="color:white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
            //         </div>
            //         <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
            //         </div>
            //         </body>
            //         </html>
            //     """
            //     emailext(
            //         subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
            //         body: body,
            //         to: 'zholshygulnaz@gmail.com',
            //         from: 'jenkins@example.com',
            //         replyTo: 'jenkins@example.com',
            //         mimeType: 'text/html',
            //         attachmentsPattern: 'trivy-report.html'
            //     )
            // }
            emailext body: 'jenkins success or failure', subject: 'test', to: 'zholshygulnaz@gmail.com'
        }
    }
}

def COLOR_MAP = [
    'SUCCESS': '00FF00', 
    'FAILURE': 'FF0000',
    'UNSTABLE': 'FFFF00'
]
pipeline{
    agent any

    triggers {
        githubPush()
    }

    tools {
        jdk 'JDK17' 
        maven 'MAVEN3.9' 
    }

    environment {
        // Replace 'your-dockerhub-user' with your actual Docker Hub username
        DOCKER_USER = 'aydin3008'
        DOCKER_REPO = 'vprofileapp'
        DOCKER_IMAGE = "${DOCKER_USER}/${DOCKER_REPO}"
        // This must match the ID you created in Step 1
        DOCKER_CREDS_ID = 'dockertoken'
    }

    stages{
        stage('Fetch Code'){
            steps {
                echo 'Fetching code...'
                checkout scm 
            }        }
        stage('Build'){
            steps{
                echo 'Building...'
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Build successful!'
                    echo 'Archiving artifacts...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps{
                echo 'Testing...'
                sh 'mvn test'
            }
        }
        stage('Checkstyle'){
            steps{
                echo 'Testing...'
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage("Sonar Code Analysis") {
        	environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
              withSonarQubeEnv('sonarserver') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        // stage("UploadArtifacttoNexus") {
        //     steps{
        //         nexusArtifactUploader(
        //           nexusVersion: 'nexus3',
        //           protocol: 'http',
        //           nexusUrl: 'nexus:8081',
        //           groupId: 'QA',
        //           version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        //           repository: 'vprofile-repo',
        //           credentialsId: 'nexuslogin',
        //           artifacts: [
        //             [artifactId: 'vproapp',
        //              classifier: '',
        //              file: 'target/vprofile-v2.war',
        //              type: 'war']
        //           ]
        //         )
        //     }
        // }
        stage('Build Docker Image') {
            steps {
                script {
                    // This builds the image using the multistage Dockerfile in your repo
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}", "-f ./Docker-files/app/multistage/Dockerfile .")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Jenkins handles the 'docker login' automatically using these credentials
                    docker.withRegistry('', DOCKER_CREDS_ID) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Local Docker') {
            steps {
                script {
                sh "docker pull ${DOCKER_IMAGE}:latest" 
                sh "docker rm -f vprofile-app || true"
                sh "docker run -d --name vprofile-app -p 8082:8080 ${DOCKER_IMAGE}:latest"                
                }
            }
        }
    }
    post {
            always {
                echo 'Discord Notifications.'
                discordSend description: "Job: ${env.JOB_NAME} \nBuild: ${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}",
                            footer: "Jenkins DevSecOps Lab",
                            link: env.BUILD_URL,
                            result: currentBuild.currentResult,
                            title: "Build Result: ${currentBuild.currentResult}",
                            webhookURL: "https://discord.com/api/webhooks/1477522408641662988/PeqCcObvfYx7d571HD-vdFYJ8cqcpkwvzW-dA4nG8kP_L-8BAS4f4kUnKQyZdKWTzFh7"
            }
            cleanup {
                sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest || true"
            }
        }
}
   
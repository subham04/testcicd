pipeline {
    agent any

    tools {
        maven 'Maven_3'  // Use the Maven tool configured in Jenkins
    }

    environment {
        ARTIFACTORY_URL = 'https://your-artifactory-server/artifactory'
        ARTIFACTORY_REPO = 'libs-release-local'
        SLACK_WEBHOOK = 'https://hooks.slack.com/services/your/webhook'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/subham04/maventest.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server 'artifactory-id'  // Artifactory credential ID in Jenkins
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "target/*.jar",
                            "target": "${ARTIFACTORY_REPO}/"
                        }]
                    }"""
                    server.upload spec: uploadSpec
                }
            }
        }

        stage('Notify via Slack') {
            steps {
                script {
                    def message = "Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} [#${env.BUILD_NUMBER}](${env.BUILD_URL})"
                    sh "curl -X POST --data-urlencode 'payload={\"text\": \"${message}\"}' ${SLACK_WEBHOOK}"
                }
            }
        }
    }

    post {
        success {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Build Successful: ${env.JOB_NAME}",
                 body: "Build ${env.BUILD_NUMBER} was successful. Check Jenkins for details."
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Build Failed: ${env.JOB_NAME}",
                 body: "Build ${env.BUILD_NUMBER} failed. Check Jenkins logs for errors."
        }
    }
}


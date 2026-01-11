pipeline {
    agent any

    stages {

        stage('Test') {
            steps {
                sh 'gradle test'
                junit 'build/test-results/test/*.xml'
                sh 'gradle cucumber'
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'gradle sonarqube'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                sh 'gradle build'
                sh 'gradle javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar'
                archiveArtifacts artifacts: 'build/docs/javadoc/**'
            }
        }

        stage('Deploy') {
            steps {
                sh 'gradle publish'
            }
        }
    }

    post {
        success {
            mail to: "team@tp7.com",
                 subject: "TP7 deployed",
                 body: "Deployment success"
        }
        failure {
            mail to: "team@tp7.com",
                 subject: "TP7 failed",
                 body: "Pipeline failed"
        }
    }
}

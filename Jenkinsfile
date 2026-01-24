pipeline {
     agent { label 'build' }

    environment {
        MAVEN_USER = credentials('maven-user')
        MAVEN_PASSWORD = credentials('maven-password')
    }

    stages {

        stage('Test') {
            steps {
                echo "Lancement des tests unitaires"
                bat './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/**/*.xml'
                }
            }
        }

        stage('Generate HTML report') {
            steps {
                echo "Génération du rapport Cucumber"
                // Assurez-vous que plugin Cucumber est installé
                cucumber(
                    buildStatus: 'UNSTABLE',
                    reportTitle: 'Cucumber Test Report',
                    fileIncludePattern: '**/cucumber*.json',
                    trendsLimit: 10
                )
            }
        }

        stage('Analyse du Code') {
            steps {
                echo "Analyse SonarQube"
                withSonarQubeEnv('sonar') {
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    echo "Vérification du Quality Gate"
                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Quality Gate échoué : ${qg.status}"
                        }
                        echo "✅ Quality Gate validé"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo "Construction du projet"
                bat './gradlew clean build'
                bat './gradlew javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: 'build/docs/javadoc/**'
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement vers Maven Repository"
                withCredentials([
                    usernamePassword(credentialsId: 'maven-user', usernameVariable: 'MVN_USER', passwordVariable: 'MVN_PWD')
                ]) {
                    bat """
                        ./gradlew publish ^
                        -PMAVEN_USER=%MVN_USER% ^
                        -PMAVEN_PASSWORD=%MVN_PWD%
                    """
                }
            }
        }
//slack notification step
        stage('Notification') {
            steps {
                slackSend(
                    teamDomain: 'jenkins-yyg2034',  // nom exact du workspace
                    channel: '#tp-jenkins',          // channel existant
                    message: '🚀 Déploiement réussi',
                    color: 'good',
                    tokenCredentialId: 'slack-bot-token'
                )
            }
        }

    }

    post {
        always {
            echo "Pipeline terminé"
        }

        success {
            emailext(
                subject: "✅ Build réussi - Jenkins",
                body: "Le pipeline Jenkins s'est exécuté avec succès.",
                to: "mb_doulateserouri@esi.dz"
            )
        }

        failure {
            slackSend(
                teamDomain: 'jenkins-yyg2034',
                channel: '#tp-jenkins',
                message: '❌ Pipeline Jenkins échoué',
                color: 'danger',
                tokenCredentialId: 'slack-bot-token'
            )
        }
    }
}

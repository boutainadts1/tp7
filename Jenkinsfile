pipeline {
    agent any

    environment {
        MAVEN_USER = credentials('maven-user')
        MAVEN_PASSWORD = credentials('maven-password')
    }

    stages {

        /* ===================== TEST ===================== */
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

        /* ===================== CUCUMBER REPORT ===================== */
        stage('Generate HTML report') {
            steps {
                echo "Génération du rapport Cucumber"
                cucumber(
                    buildStatus: 'UNSTABLE',
                    reportTitle: 'Cucumber Test Report',
                    fileIncludePattern: '**/cucumber*.json',
                    trendsLimit: 10
                )
            }
        }

        /* ===================== CODE ANALYSIS ===================== */
        stage('Analyse du Code') {
            steps {
                echo "Analyse SonarQube"
                withSonarQubeEnv('sonar') {
                    bat './gradlew sonarqube'
                }
            }
        }

        /* ===================== QUALITY GATE ===================== */
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

        /* ===================== BUILD ===================== */
        stage('Build') {
            steps {
                echo "Construction du projet"
                bat './gradlew clean build'
                bat './gradlew javadoc'

                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: 'build/docs/javadoc/**'
            }
        }

        /* ===================== DEPLOY ===================== */
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


        /* ===================== NOTIFICATION ===================== */
        stage('Notification') {
            steps {
                slackSend(
                    channel: '#tp-jenkins',
                    message: '🚀 Déploiement réussi',
                    color: 'good',
                    tokenCredentialId: 'slack-bot-token'
                )
            }
        }
    }

    /* ===================== POST ACTIONS ===================== */
    post {
        always {
            echo "Pipeline terminé"
        }

        success {
            emailext(
                subject: "✅ Build réussi - Jenkins",
                body: "Le pipeline Jenkins s'est exécuté avec succès.",
                to: "doulateserouriboutaina@gmail.com"
            )
        }

        failure {
            slackSend(
                channel: '#general',
                message: '❌ Pipeline Jenkins échoué',
                color: 'danger',
                tokenCredentialId: 'slack-bot-token'
            )
        }
    }
}

pipeline {
    agent any

    environment {
        SLACK_WEBHOOK_URL = credentials('slack-webhook')    //slack-webhook-url
        GIT_CREDENTIALS_ID = 'token_clase'  //git-credentials-id
        GIT_REPO_URL = 'https://github.com/Andres-Vazquez-Leon/practica_ci_cd_v2.git'
        MAIN_BRANCH = 'master'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    if (env.BRANCH_NAME == MAIN_BRANCH) {
                        error "No se puede hacer merge desde la rama principal a sí misma."
                    }
                    checkout scm
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh 'mvn test'
                    } catch (Exception e) {
                        notifySlack("❌ Pruebas fallidas en la rama ${env.BRANCH_NAME}")
                        notifyEmail("Pruebas fallidas en la rama ${env.BRANCH_NAME}")
                        error "Pruebas fallidas. Build marcado como fallido."
                    }
                }
            }
        }

        stage('Merge to Master') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh """
                            git config user.name "Andres-Vazquez-Leon"
                            git config user.email "andres.vazquezleon01@gmail.com"
                            git checkout ${MAIN_BRANCH}
                            git pull origin ${MAIN_BRANCH}
                            git merge --no-ff ${env.BRANCH_NAME} -m "Merge automático desde ${env.BRANCH_NAME}"
                            git push https://${GIT_USER}:${GIT_PASS}@${GIT_REPO_URL} ${MAIN_BRANCH}
                        """
                    }
                    notifySlack("✅ Merge exitoso de ${env.BRANCH_NAME} a ${MAIN_BRANCH}")
                    notifyEmail("Merge exitoso de ${env.BRANCH_NAME} a ${MAIN_BRANCH}")
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completado exitosamente."
        }
        failure {
            echo "Pipeline fallido."
        }
        always {
            echo "Pipeline terminado."
        }
    }
}

def notifySlack(message) {
    try {
        sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text": "${message}", "channel": "#alertas"}' \
            "${SLACK_WEBHOOK_URL}"
        """
    } catch (Exception e) {
        echo "⚠️ No se pudo enviar notificación a Slack: ${e.getMessage()}"
    }
}

def notifyEmail(message) {
    try {
        mail to: 'andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com',
             subject: message,
             body: "Este es un mensaje automático de Jenkins."
    } catch (Exception e) {
        echo "⚠️ No se pudo enviar notificación por correo: ${e.getMessage()}"
    }
}

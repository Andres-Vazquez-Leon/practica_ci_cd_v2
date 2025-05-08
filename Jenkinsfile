pipeline {
    agent any

    environment {
        SLACK_WEBHOOK_URL = credentials('slack-webhook')
        GIT_CREDENTIALS_ID = 'token_clase'
        GIT_REPO_URL = 'https://github.com/Andres-Vazquez-Leon/practica_ci_cd_v2.git'
        MAIN_BRANCH = 'master'
        EMAIL_RECIPIENTS = 'andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Verifica si es la rama principal para evitar loops de merge
                    if (env.BRANCH_NAME == MAIN_BRANCH) {
                        error "❌ No se puede hacer merge desde la rama principal a sí misma."
                    }
                    // Clona el repositorio
                    checkout scm
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    try {
                        // Ajusta este comando según tu stack de pruebas
                        sh 'mvn test'
                    } catch (Exception e) {
                        notifySlack("❌ Pruebas fallidas en la rama ${env.BRANCH_NAME}")
                        notifyEmail("❌ Pruebas fallidas en la rama ${env.BRANCH_NAME}")
                        error "Pruebas fallidas. Build marcado como fallido."
                    }
                }
            }
        }

        stage('Merge to Master') {
            when {
                expression { env.BRANCH_NAME != MAIN_BRANCH }
            }
            steps {
                script {
                    // Realiza el merge usando Git
                    sh """
                        git config user.name "Andres-Vazquez-Leon"
                        git config user.email "andres.vazquezleon01@gmail.com"
                        git checkout ${MAIN_BRANCH}
                        git pull origin ${MAIN_BRANCH}
                        git merge --no-ff ${env.BRANCH_NAME} -m "Merge automático desde ${env.BRANCH_NAME}"
                        git push origin ${MAIN_BRANCH}
                    """
                    notifySlack("✅ Merge exitoso de ${env.BRANCH_NAME} a ${MAIN_BRANCH}")
                    notifyEmail("✅ Merge exitoso de ${env.BRANCH_NAME} a ${MAIN_BRANCH}")
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completado exitosamente."
        }
        failure {
            echo "❌ Pipeline fallido."
        }
        always {
            echo "🔁 Pipeline terminado."
        }
    }
}

def notifySlack(message) {
    try {
        sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text": "${message}", "channel": "#alertas"}' \
            ${SLACK_WEBHOOK_URL}
        """
    } catch (Exception e) {
        echo "⚠️ No se pudo enviar notificación a Slack: ${e.getMessage()}"
    }
}

def notifyEmail(message) {
    try {
        mail to: env.EMAIL_RECIPIENTS,
             subject: message,
             body: "Este es un mensaje automático de Jenkins."
    } catch (Exception e) {
        echo "⚠️ No se pudo enviar notificación por correo: ${e.getMessage()}"
    }
}

pipeline {
    agent any

    environment {
        NUCLEI_SCAN_PATH = 'nuclei'  // Assurez-vous que Nuclei est installé ou qu'il est disponible via Docker
        SERVER_PORT = '3000'         // Port pour l'application Node.js
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main', url: 'https://github.com/korgrunt/vuln-server' // Remplace par l'URL du dépôt
            }
        }

        stage('Instancier le serveur Node.js') {
            steps {
                script {
                    echo 'Démarrage de l\'application Node.js...'
                    // Lancer le serveur Node.js en arrière-plan
                    sh 'nohup node server.js &'
                    // Attendre un peu pour s'assurer que le serveur est bien lancé
                    sleep 10
                }
            }
        }

        stage('Exécution de Nuclei') {
            steps {
                script {
                    echo 'Lancement du scan dynamique avec Nuclei...'

                    // Exécuter Nuclei sur l'application Node.js en local
                    sh """
                        ${NUCLEI_SCAN_PATH} -u http://localhost:${SERVER_PORT} -o report-nuclei.txt
                    """
                }
            }
        }

        stage('Analyse des résultats') {
            steps {
                script {
                    echo 'Analyse des résultats de Nuclei...'

                    // Lire les résultats du fichier de rapport généré par Nuclei
                    def report = readFile('report-nuclei.txt')

                    // Vérifier si des vulnérabilités high, medium ou low ont été trouvées
                    if (report.contains("high") || report.contains("medium") || report.contains("low")) {
                        echo 'Vulnérabilités détectées dans le scan Nuclei, échec du build.'
                        currentBuild.result = 'FAILURE'
                    } else {
                        echo 'Aucune vulnérabilité détectée, build réussi.'
                    }
                }
            }
        }

        stage('Nettoyage') {
            steps {
                script {
                    echo 'Arrêt du serveur Node.js...'
                    // Arrêter le serveur Node.js si nécessaire (en fonction du système d'exploitation)
                    sh 'pkill -f node'
                }
            }
        }
    }

    post {
        success {
            echo 'Build terminé avec succès. Aucune vulnérabilité détectée.'
        }
        failure {
            echo 'Le build a échoué en raison de vulnérabilités détectées par Nuclei.'
        }
        always {
            cleanWs()  // Nettoyer le workspace après l'exécution
        }
    }
}

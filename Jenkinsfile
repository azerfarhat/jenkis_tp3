pipeline {
    agent any
    options { timestamps() }

    environment {
        IMAGE = 'azerfarhat/tpdevops'   // Nom de ton image Docker Hub
        TAG = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git url: 'https://github.com/azerfarhat/jenkis_tp3.git', branch: 'main'
            }
        }

        stage('Vérification du dépôt') {
            steps {
                bat 'echo === Vérification du dépôt ==='
                bat '"C:\\Program Files\\Git\\bin\\git.exe" status'
            }
        }

        stage('Afficher le contenu du projet') {
            steps {
                bat 'echo === Contenu du projet ==='
                bat 'dir'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'echo === Étape : Construction de l\'image Docker ==='
                bat 'docker version'
                bat "docker build -t %IMAGE%:%TAG% ."
            }
        }

        stage('Smoke Test (test rapide)') {
            steps {
                bat 'echo === Étape : Test rapide de l\'image ==='
                bat """
                docker rm -f monapp_test 2>nul || ver > nul
                docker run -d --name monapp_test -p 8085:80 %IMAGE%:%TAG%
                ping -n 5 127.0.0.1 > nul
                curl -I http://localhost:8085 | find "200 OK"
                docker rm -f monapp_test
                """
            }
        }

        stage('Push vers Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS')]) {
                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker tag %IMAGE%:%TAG% %IMAGE%:latest
                    docker push %IMAGE%:%TAG%
                    docker push %IMAGE%:latest
                    """
                }
            }
        }

        stage('Fin du pipeline') {
            steps {
                bat 'echo === Pipeline terminé avec succès ==='
            }
        }
    }

    post {
        success { echo '✅ Build + Test + Push OK' }
        failure { echo '❌ Échec du pipeline' }
    }
}

pipeline {
    agent any
    tools {
        maven 'Maven3'   // Nom du Maven configuré dans Jenkins Global Tool Configuration
        jdk 'Java17'     // Nom du JDK configuré dans Jenkins Global Tool Configuration
    }
    environment {
        SONAR = 'SonarQube'        // Nom du serveur SonarQube dans Configure System
        NEXUS_CRED = 'nexus-admin' // Credential Jenkins pour Nexus
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPO = 'maven-releases'
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '10')) // Gérer anciens builds
        timestamps() // Affiche les timestamps dans la console
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Clonage du code depuis GitHub"
                git branch: 'main', url: 'https://github.com/Michella20/demoic.git', credentialsId: 'github-jenkins'
            }
        }
        stage('Build & Test') {
            steps {
                echo "Compilation et tests Maven"
                sh 'mvn clean verify'
            }
        }
        stage('Analyse SonarQube') {
            steps {
                echo "Analyse SonarQube"
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Publish Artifact to Nexus') {
            steps {
                echo "Publication sur Nexus"
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${env.NEXUS_URL}",
                    repository: "${env.NEXUS_REPO}",
                    credentialsId: "${env.NEXUS_CRED}",
                    groupId: 'com.demoic',
                    version: '1.0.0',
                    artifactId: 'demoic',
                    file: 'target/demoic.jar',
                    packaging: 'jar'
                )
            }
        }
    }
    post {
        success {
            echo "Pipeline terminé avec succès !"
        }
        failure {
            echo "Le pipeline a échoué. Vérifie la console pour plus de détails."
        }
        always {
            echo "Fin du pipeline."
        }
    }
}
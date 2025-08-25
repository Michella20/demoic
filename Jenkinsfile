pipeline {
    agent any

    tools {
        jdk 'java'         // Nom exact du JDK 17 configuré dans Jenkins Global Tool Configuration
        maven 'Maven3'       // Nom exact de Maven configuré dans Jenkins
    }

    environment {
        SONAR = 'SonarQube'           // Nom du serveur SonarQube dans Jenkins Configure System
        NEXUS_CRED = 'nexus-admin'    // Credential Jenkins pour Nexus
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPO = 'maven-releases'
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '10')) // garder 30 jours ou 10 builds
        timestamps() // affichage timestamps dans la console
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning code from GitHub"
                git branch: 'main',
                    url: 'https://github.com/Michella20/demoic.git',
                    credentialsId: 'github-jenkins'
            }
        }

        stage('Build & Test') {
            steps {
                echo "Running Maven build and tests"
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Publish Artifact to Nexus') {
            steps {
                echo "Uploading artifact to Nexus"
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${env.NEXUS_URL}",
                    repository: "${env.NEXUS_REPO}",
                    credentialsId: "${env.NEXUS_CRED}",
                    groupId: 'com.demoic',
                    artifactId: 'demoic',
                    version: '1.0.0',
                    file: 'target/demoic.jar',
                    packaging: 'jar'
                )
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully!"
        }
        failure {
            echo "Pipeline failed. Check console output for details."
        }
        always {
            echo "End of Pipeline."
        }
    }
}

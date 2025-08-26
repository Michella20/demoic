pipeline {
    agent any

    tools {
        jdk 'java'         // Nom exact du JDK 17 configuré dans Jenkins Global Tool Configuration
        maven 'Maven3'     // Nom exact de Maven configuré dans Jenkins
    }

    environment {
        SONAR = 'SonarQube'           // Nom du serveur SonarQube dans Jenkins Configure System
        NEXUS_CRED = 'nexus-admin'    // Credential Jenkins pour Nexus
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPO = 'maven-releases'
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '10'))
        timestamps()
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

        stage('Test Tools') {
            steps {
                echo "Testing if CMD, Java, and Maven are available"
                bat 'echo Hello from Windows CMD'
                bat 'where java'
                bat 'where mvn'
            }
        }

        stage('Build & Test') {
            steps {
                echo "Running Maven build and tests"
                bat 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv('SonarQube') {
                    bat 'mvn sonar:sonar'
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
                    artifacts: [
                        [
                            artifactId: 'demoic',
                            classifier: '',
                            file: 'target/demoic.jar',
                            type: 'jar'
                        ]
                    ],
                    version: '1.0.0'
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

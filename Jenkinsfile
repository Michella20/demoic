pipeline {
    agent any

    environment {
        // Chemins complets pour cmd, java et maven
        CMD = 'C:\\Windows\\System32\\cmd.exe'
        JAVA_HOME = 'C:\\Users\\Laptop_Pavilion\\Downloads\\OpenJDK17U-jdk_x64_windows_hotspot_17.0.16_8\\jdk-17.0.16+8'
        MAVEN_HOME = 'C:\\Users\\Laptop_Pavilion\\Downloads\\apache-maven-3.9.11-bin\\apache-maven-3.9.11'
        PATH = "${env.JAVA_HOME}\\bin;${env.MAVEN_HOME}\\bin;${env.PATH}"

        SONAR = 'SonarQube'
        NEXUS_CRED = 'nexus-admin'
        NEXUS_URL = 'localhost:8081'    // <- Retirer le "http://" ici
        NEXUS_REPO = 'maven-snapshots'
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
                echo "Test if CMD, Java and Maven are available"
                bat "${CMD} /c echo CMD is available"
                bat "${CMD} /c \"${JAVA_HOME}\\bin\\java -version\""
                bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn -version\""

            }
        }

        stage('Build & Test') {
            steps {
                echo "Running Maven build and tests"
                bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn clean verify\""
		
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv('SonarQube') {
                    bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn sonar:sonar\""
                }
            }
        }

        stage('Publish Artifact to Nexus') {
            steps {
 		script {
	           def artifactId = 'demoic'
                   def version = '1.0-SNAPSHOT'
                   def jarFile = "target/${artifactId}-${version}.jar"
            	echo "Publishing JAR: ${jarFile}"
                echo "Uploading artifact to Nexus"
		 
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',                     // <- Protocole séparé
                    nexusUrl: "${env.NEXUS_URL}",         // <- Sans "http://"
                    repository: "${env.NEXUS_REPO}",
                    credentialsId: "${env.NEXUS_CRED}",
                    groupId: 'com.demoic',
                    version: 'version',
                    artifacts: [
                        [
                            artifactId: 'artifactId',
                            classifier: '',
                            file: 'jarFile',
                            type: 'jar'
                        ]
                    ]
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

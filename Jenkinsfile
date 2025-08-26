pipeline {
    agent any

    environment {
        CMD = 'C:\\Windows\\System32\\cmd.exe'
        JAVA_HOME = 'C:\\Users\\Laptop_Pavilion\\Downloads\\OpenJDK17U-jdk_x64_windows_hotspot_17.0.16_8\\jdk-17.0.16+8'
        MAVEN_HOME = 'C:\\Users\\Laptop_Pavilion\\Downloads\\apache-maven-3.9.11-bin\\apache-maven-3.9.11'
        PATH = "${env.JAVA_HOME}\\bin;${env.MAVEN_HOME}\\bin;${env.PATH}"

        SONAR = 'SonarQube'
        NEXUS_CRED = 'nexus-admin'
        NEXUS_URL = 'localhost:8081'
        NEXUS_RELEASE_REPO = 'maven-releases'
        NEXUS_SNAPSHOT_REPO = 'maven-snapshots'
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

        stage('Build & Test SNAPSHOT') {
            steps {
                echo "Building SNAPSHOT version"
                bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn clean verify\""
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv(SONAR) {
                    bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn sonar:sonar\""
                }
            }
        }

        stage('Publish SNAPSHOT to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def snapshotVersion = pom.version
                    echo "Publishing SNAPSHOT ${snapshotVersion} to ${env.NEXUS_SNAPSHOT_REPO}"

                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: env.NEXUS_URL,
                        repository: env.NEXUS_SNAPSHOT_REPO,
                        credentialsId: env.NEXUS_CRED,
                        groupId: 'com.demoic',
                        artifacts: [
                            [
                                artifactId: 'demoic',
                                classifier: '',
                                file: "target/demoic-${snapshotVersion}.jar",
                                type: 'jar'
                            ]
                        ],
                        version: snapshotVersion
                    )
                }
            }
        }

        stage('Build & Publish RELEASE') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def releaseVersion = pom.version.replace('-SNAPSHOT','')
                    echo "Building RELEASE version ${releaseVersion}"

                    // Changer la version Maven à RELEASE
                    bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn versions:set -DnewVersion=${releaseVersion}\""
                    bat "${CMD} /c \"${MAVEN_HOME}\\bin\\mvn clean verify\""

                    echo "Publishing RELEASE ${releaseVersion} to ${env.NEXUS_RELEASE_REPO}"
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: env.NEXUS_URL,
                        repository: env.NEXUS_RELEASE_REPO,
                        credentialsId: env.NEXUS_CRED,
                        groupId: 'com.demoic',
                        artifacts: [
                            [
                                artifactId: 'demoic',
                                classifier: '',
                                file: "target/demoic-${releaseVersion}.jar",
                                type: 'jar'
                            ]
                        ],
                        version: releaseVersion
                    )
                }
            }
        }
    }

    post {
        success { echo "Pipeline finished successfully!" }
        failure { echo "Pipeline failed. Check console output." }
        always { echo "End of Pipeline." }
    }
}

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
 
    stages {
        stage('Source') {
            steps {
                git branch: 'main', url: 'https://github.com/mdabuateef/jenkins-war.git'
            }
        }
 
        stage('Build') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withMaven {
                        sh 'mvn clean verify sonar:sonar'
                    }
                }
            }
        }
        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: '65.0.89.234:8081/', // Corrected URL
                            groupId: 'pom.com.mt',
                            version: 'pom.2.0',
                            repository: 'abuateef',
                            credentialsId: 'abuateef',
                            artifacts: [
                                [
                                    artifactId: 'pom.java-web-app',
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: 'pom.java-web-app',
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
        stage('Pull Artifact from Nexus') {
            steps {
                script {
                    def nexusRepoUrl = 'http://65.0.89.234:8081/repository/abuateef/'
                    def groupId = 'pom.com.mt'
                    def artifactId = 'pom.java-web-app'
                    def version = 'pom.2.0'
                    def packaging = 'war'
 
                    // Download the artifact
                    sh "curl -O ${nexusRepoUrl}/${groupId}/${artifactId}/${version}/${artifactId}-${version}.${packaging}"
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                script {
                    def artifactId = 'pom.java-web-app'
                    def version = 'pom.2.0'
                    def packaging = 'war'
                    def tomcatWebappsDir = '/opt/tomcat/webapps'
 
                    // Copy the downloaded artifact to the Tomcat webapps directory
                    sh 'sudo rm -r /opt/tomcat/webapps/*.war'
                    sh "sudo cp ${artifactId}-${version}.${packaging} ${tomcatWebappsDir}/"
                }
            }
        }
    }
}

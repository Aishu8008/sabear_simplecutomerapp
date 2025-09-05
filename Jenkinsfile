pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "18.232.50.163:8081/"
        NEXUS_REPOSITORY = "Hiring_app"
        NEXUS_CREDENTIAL_ID = "nexus"
        SCANNER_HOME = tool 'sonar_scanner'
        // Slack details (already configured in Jenkins → Configure System → Slack)
        SLACK_CHANNEL = "#jenkins-integration"
    }
    stages {
        stage("clone code") {
            steps {
                git 'https://github.com/Aishu8008/sabear_simplecutomerapp.git'
            }
        }
        stage("mvn build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }
        stage("SonarCloud") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner\
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
                        -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=src/com/room/sample '''
                }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    
        stage("Deploy to Tomcat") {
    steps {
        withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            script {
                // Find the WAR file built by Maven
                def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."
                sh """
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T ${warFile} \
                         "http://44.210.239.111/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }
        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#36A64F",
                    message: "Declarative pipeline for *Simple Customer App* has been successfully deployed in Tomcat :white_check_mark: by SNL for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
    }
}

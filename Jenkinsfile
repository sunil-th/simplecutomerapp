pipeline {
    agent any
    tools {
        maven "MVN_HOME"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.83.214.6:8081"
        NEXUS_REPOSITORY = "pipe-snapshots"
        NEXUS_CREDENTIAL_ID = "Nexus-server"
        SCANNER_HOME = tool 'sonar_scanner'

        // Slack details (already configured in Jenkins → Configure System → Slack)
        SLACK_CHANNEL = "#jenkins-integration"

        // Tomcat details
        TOMCAT_SERVER = "ec2-user@3.89.121.33"   // ✅ no angle brackets
        TOMCAT_PATH = "/opt/tomcat/webapps"      // Tomcat webapps folder
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    git 'https://github.com/sunil-th/simplecutomerapp.git'
                }
            }
        }

        stage("mvn build") {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }

        stage('SonarCloud') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
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

        stage("Slack Notification") {
            steps {
                script {
                    slackSend(
                        channel: "${SLACK_CHANNEL}",
                        color: "#36a64f",
                        message: "✅ Build & Nexus Upload Successful for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                script {
                    echo "Deploying WAR to Tomcat..."
                    sshagent(['tomcat']) {   // ✅ replace 'tomcat' with your Jenkins credentialsId
                        sh """
                            scp -o StrictHostKeyChecking=no target/*.war ${TOMCAT_SERVER}:${TOMCAT_PATH}/
                        """
                    }
                }
            }
        }
    }
    post {
        failure {
            script {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#ff0000",
                    message: "❌ Build Failed for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
    }
}

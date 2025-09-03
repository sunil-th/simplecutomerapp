pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "MVN_HOME"
        
    }
	 environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "3.83.214.6:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "sonarqube-simple-app"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "Nexus-server"
	SCANNER_HOME = tool 'sonar_scanner'
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/sunil-th/simplecutomerapp.git';
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }
	stage('SonarCloud') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
				sh '$SCANNER_HOME/bin/sonar-scanner \
				-Dsonar.projectKey=Ncodeit \
				-Dsonar.projectName=Ncodeit \
				-Dsonar.projectVersion=2.0 \
				-Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
				-Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
				-Dsonar.junit.reportsPath=target/surefire-reports \
				-Dsonar.jacoco.reportPath=target/jacoco.exec \
				-Dsonar.java.binaries=src/com/room/sample '
				
		     }
		}
	    }
        stage("publish to nexus") {
            steps {
                script {
    // Parse pom.xml with XmlSlurper instead of readMavenPom
    def pomText = readFile('pom.xml')
    def pomXml = new XmlSlurper().parseText(pomText)

    def groupId    = pomXml.groupId.text()
    def artifactId = pomXml.artifactId.text()
    def version    = pomXml.version.text()
    def packaging  = pomXml.packaging.text()

    // Find built artifact under target folder
    def filesByGlob = findFiles(glob: "target/*.${packaging}")

    // Print some info from the artifact found
    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"

    // Extract the path from the File found
    def artifactPath = filesByGlob[0].path

    // Assign to a boolean response verifying if the artifact name exists
    def artifactExists = fileExists artifactPath

    if (artifactExists) {
        echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version ${version}"

        nexusArtifactUploader(
            nexusVersion: NEXUS_VERSION,
            protocol: NEXUS_PROTOCOL,
            nexusUrl: NEXUS_URL,
            groupId: groupId,
            version: version,
            repository: NEXUS_REPOSITORY,
            credentialsId: NEXUS_CREDENTIAL_ID,
            artifacts: [
                // Artifact generated such as .jar, .ear and .war files.
                [artifactId: artifactId,
                 classifier: '',
                 file: artifactPath,
                 type: packaging],
                // Upload pom.xml file for dependency resolution
                [artifactId: artifactId,
                 classifier: '',
                 file: "pom.xml",
                 type: "pom"]
            ]
        )
    } else {
        error "*** File: ${artifactPath}, could not be found"
    }
}

            }
        }
    }
}

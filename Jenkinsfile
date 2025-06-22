pipeline {
    agent any

    environment {
        ARTIFACT_ID = 'elearn-web'
        VERSION = '1.0.0'
        FILE_PATH = "target/${ARTIFACT_ID}-${VERSION}.war"
        GROUP_ID = 'in.karthik'
        REPO = 'webapp-releases'
        NEXUS_URL = '192.168.56.102:8081'
        CREDENTIALS_ID = 'nexus3'
        TOMCAT_IP = '192.168.56.102'
        TOMCAT_WEBAPPS = '/opt/tomcat/webapps'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev1', url: 'https://github.com/Karthikeyareddy81/ELEARN-WEB.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload WAR to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: "${ARTIFACT_ID}",
                        classifier: '',
                        file: "${FILE_PATH}",
                        type: 'war'
                    ]
                ],
                credentialsId: "${CREDENTIALS_ID}",
                groupId: "${GROUP_ID}",
                nexusUrl: "${NEXUS_URL}",
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: "${REPO}",
                version: "${VERSION}"
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                sshagent (credentials: ['tomcat-ssh']) {
                    sh """
                    scp ${FILE_PATH} root@${TOMCAT_IP}:/tmp/
                    ssh root@${TOMCAT_IP} <<EOF
                        rm -rf ${TOMCAT_WEBAPPS}/${ARTIFACT_ID}
                        cp /tmp/${ARTIFACT_ID}-${VERSION}.war ${TOMCAT_WEBAPPS}/
                        rm /tmp/${ARTIFACT_ID}-${VERSION}.war
                    EOF
                    """
                }
            }
        }
    }
}

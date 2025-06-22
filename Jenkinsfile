pipeline {
    agent any

    environment {
        ARTIFACT_ID = 'elearn-web'
        VERSION = '1.0.0'
        FILE_NAME = "${ARTIFACT_ID}-${VERSION}.war"
        FILE_PATH = "target/${FILE_NAME}"
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
                echo '🔨 Building WAR file...'
                sh 'mvn clean package'
                sh 'ls -l target/' // for visibility in Jenkins logs
            }
        }

        stage('Upload WAR to Nexus') {
            steps {
                echo "⬆️ Uploading ${FILE_NAME} to Nexus..."
                nexusArtifactUploader artifacts: [[
                    artifactId: "${ARTIFACT_ID}",
                    classifier: '',
                    file: "${FILE_PATH}",
                    type: 'war'
                ]],
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
                echo '🚀 Deploying WAR to Tomcat...'
                sshagent (credentials: ['tomcat-ssh']) {
                    sh """
                    echo '📦 Copying WAR to Tomcat server...'
                    scp ${FILE_PATH} root@${TOMCAT_IP}:/tmp/

                    echo '🧹 Cleaning old deployment and redeploying...'
                    ssh root@${TOMCAT_IP} <<EOF
                        rm -rf ${TOMCAT_WEBAPPS}/${ARTIFACT_ID}
                        cp /tmp/${FILE_NAME} ${TOMCAT_WEBAPPS}/
                        rm /tmp/${FILE_NAME}
                    EOF
                    """
                }
            }
        }
    }
}

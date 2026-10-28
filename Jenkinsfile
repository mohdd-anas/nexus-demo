pipeline {
    agent any

    environment {
        PATH      = "/opt/homebrew/opt/openjdk@17/bin:/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
        NEXUS_URL = 'http://localhost:8081'
        ARTIFACT  = 'demo-app'
        VERSION   = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B clean package'
            }
        }

        stage('Upload to Nexus - curl (raw repo)') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials',
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        curl -f -u "$NEXUS_USER:$NEXUS_PASS" \
                          --upload-file target/${ARTIFACT}-${VERSION}.jar \
                          ${NEXUS_URL}/repository/raw-artifacts/${ARTIFACT}/${VERSION}/${ARTIFACT}-${VERSION}.jar
                    '''
                }
            }
        }

        stage('Deploy to Nexus - mvn deploy (maven repo)') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials',
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    sh 'mvn -B -s settings.xml deploy -DskipTests'
                }
            }
        }
    }

    post {
        success { echo 'Artifacts uploaded to Nexus successfully.' }
        failure { echo 'Pipeline failed. Check the console output.' }
    }
}

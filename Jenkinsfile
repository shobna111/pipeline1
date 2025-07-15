pipeline {
    agent any

    tools {
        maven 'M3' // Jenkins global tool config
        jdk 'jdk17'
    }

    environment {
        // Nexus
        NEXUS_URL = 'http://18.207.125.168:8081'
        NEXUS_REPO = 'releases'
        GROUP_ID = 'com/example/app'    // adjust as needed
        ARTIFACT_ID = 'basic-java-app'
        VERSION = '1.0-SNAPSHOT'

        // Deployment server
        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '18.207.125.168'
        DEPLOY_PATH = '/var/www/html'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                    ARTIFACT=target/${ARTIFACT_ID}-${VERSION}.jar
                    curl -v -u $NEXUS_USER:$NEXUS_PASS --upload-file $ARTIFACT \
                    $NEXUS_URL/repository/$NEXUS_REPO/${GROUP_ID}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.jar
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'nginx-ssh-creds', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                    ARTIFACT=target/${ARTIFACT_ID}-${VERSION}.jar
                    scp -i $SSH_KEY $ARTIFACT $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/app.jar
                    ssh -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST "pkill -f app.jar || true && nohup java -jar $DEPLOY_PATH/app.jar > $DEPLOY_PATH/app.log 2>&1 &"
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'echo "Cleaning up..."'
        }

        success {
            echo 'Build succeeded!'
        }

        failure {
            echo 'Build failed!'
        }
    }
}

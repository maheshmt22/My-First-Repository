pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        SERVER_IP = '54.159.31.22'
        SERVER_USER = 'ubuntu'
        TOMCAT_DIR = '/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/maheshmt22/newrepository.git'
            }
        }

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Upload to JFrog') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'JFROG_USER',
                        passwordVariable: 'JFROG_PASS'
                    )
                ]) {

                    sh '''
                        set -e

                        echo "Uploading WAR to JFrog..."

                        WAR_FILE=$(ls sample-app/target/*.war)
                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"

                        echo "WAR File: $WAR_FILE"
                        echo "Uploading as: $FILE_NAME"

                        curl -u $JFROG_USER:$JFROG_PASS \
                             -T "$WAR_FILE" \
                             "https://trial7n02kw.jfrog.io/artifactory/java_warfile_repo-generic-local/$FILE_NAME"

                        echo "Upload completed successfully!"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {

                sshagent(credentials: ['tomcat-ssh-key']) {

                    sh '''
                        set -e

                        echo "Starting deployment to Tomcat..."

                        WAR_FILE=$(ls sample-app/target/*.war)
                        WAR_NAME=$(basename $WAR_FILE)

                        echo "WAR File: $WAR_NAME"

                        # Copy WAR to remote server
                        scp -o StrictHostKeyChecking=no \
                        $WAR_FILE $SERVER_USER@$SERVER_IP:/tmp/

                        # Deploy application
                        ssh -o StrictHostKeyChecking=no \
                        $SERVER_USER@$SERVER_IP "

                            echo 'Stopping old deployment if exists...'

                            sudo rm -rf $TOMCAT_DIR/sample-app*
                            sudo mv /tmp/$WAR_NAME $TOMCAT_DIR/

                            echo 'Restarting Tomcat...'

                            sudo systemctl restart tomcat

                            sleep 10

                            sudo systemctl status tomcat --no-pager
                        "

                        echo "Deployment completed successfully!"
                    '''
                }
            }
        }
    }
}

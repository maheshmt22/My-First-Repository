pipeline {
    agent any

    parameters {

        choice(
        name: 'ENVIRONMENT',
        choices: ['DEV', 'QA', 'PROD'],
        description: 'Select deployment environment'
    )

        choice(
        name: 'ENVIRONMENT',
        choices: ['DEV', 'QA', 'PROD'],
        description: 'Deployment Environment'
    )
    }

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SERVER_USER = 'ubuntu'
        TOMCAT_DIR = '/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout Code') {
            steps {

                git branch: 'main',
                    url: 'https://github.com/maheshmt22/My-First-Repository.git'
            }
        }

        stage('Build') {
            steps {

                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }

                script {

                    env.WAR_FILE = sh(
                        script: "ls sample-app/target/*.war",
                        returnStdout: true
                    ).trim()

                    env.WAR_NAME = sh(
                        script: "basename ${env.WAR_FILE}",
                        returnStdout: true
                    ).trim()

                    echo "WAR File Path: ${env.WAR_FILE}"
                    echo "WAR Name: ${env.WAR_NAME}"
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

                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"

                        curl --fail -L \
                             -u $JFROG_USER:$JFROG_PASS \
                             -T "$WAR_FILE" \
                             "https://triallb6d6m.jfrog.io/artifactory/jenkinsjava-generic-local/$FILE_NAME"

                        echo "Upload completed successfully!"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {

                sshagent(credentials: ['tomcat-ssh-key']) {

                    sh """
                        set -e

                        echo "Environment: ${params.ENVIRONMENT}"
                        echo "Deploying WAR: ${WAR_NAME}"

                        scp -o StrictHostKeyChecking=no \
                        ${WAR_FILE} ${SERVER_USER}@${params.SERVER_IP}:/tmp/

                        ssh -o StrictHostKeyChecking=no \
                        ${SERVER_USER}@${params.SERVER_IP} '

                            sudo rm -rf ${TOMCAT_DIR}/sample-app*

                            sudo mv /tmp/${WAR_NAME} ${TOMCAT_DIR}/

                            sudo systemctl restart tomcat

                            sleep 10

                            sudo systemctl status tomcat --no-pager
                        '

                        echo "Deployment completed successfully!"
                    """
                }
            }
        }
    }
}

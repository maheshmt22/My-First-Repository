pipeline {
    agent any

    parameters {

        choice(
            name: 'ENVIRONMENT',
            choices: ['DEV', 'QA', 'PROD'],
            description: 'Select deployment environment'
        )

        string(
            name: 'SERVER_IP',
            description: 'Tomcat Server IP'
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
                        script: "find sample-app/target -name '*.war' | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!env.WAR_FILE) {
                        error "WAR file not found!"
                    }

                    env.WAR_NAME = sh(
                        script: "basename \"${env.WAR_FILE}\"",
                        returnStdout: true
                    ).trim()

                    echo "WAR File Path: ${env.WAR_FILE}"
                    echo "WAR Name: ${env.WAR_NAME}"
                }
            }
        }

        stage('Upload to JFrog') {
            steps {

                withCredentials([[
                    $class: 'UsernamePasswordMultiBinding',
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'JFROG_USER',
                    passwordVariable: 'JFROG_PASS'
                ]]) {

                    sh """
                        set -e

                        echo "Uploading WAR to JFrog..."

                        SAFE_JOB_NAME=\$(echo "${JOB_NAME}" | tr '/' '_')
                        FILE_NAME="\${SAFE_JOB_NAME}-${BUILD_NUMBER}-${ENVIRONMENT}.war"

                        curl --fail -L \
                            -u $JFROG_USER:$JFROG_PASS \
                            -T "${WAR_FILE}" \
                            "https://triallb6d6m.jfrog.io/artifactory/jenkinsjava-generic-local/\$FILE_NAME"

                        echo "Upload completed successfully!"
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {

                sshagent(credentials: ['tomcat-ssh-key']) {

                    sh """
                        set -e

                        echo "Environment: ${ENVIRONMENT}"
                        echo "Deploying WAR: ${WAR_NAME}"

                        scp -o StrictHostKeyChecking=no \
                            "${WAR_FILE}" ${SERVER_USER}@${SERVER_IP}:/tmp/

                        ssh -o StrictHostKeyChecking=no \
                            ${SERVER_USER}@${SERVER_IP} "
                                set -e

                                echo 'Stopping Tomcat safely...'
                                sudo systemctl stop tomcat

                                echo 'Cleaning old deployment...'
                                sudo rm -rf ${TOMCAT_DIR}/sample-app*

                                echo 'Deploying new WAR...'
                                sudo cp /tmp/${WAR_NAME} ${TOMCAT_DIR}/sample-app.war

                                echo 'Starting Tomcat...'
                                sudo systemctl start tomcat

                                sleep 10

                                echo 'Checking status...'
                                sudo systemctl status tomcat --no-pager
                            "

                        echo "Deployment completed successfully!"
                    """
                }
            }
        }
    }
}

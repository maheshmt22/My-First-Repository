pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
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
                        echo "Uploading WAR to JFrog..."

                        WAR_FILE=$(ls sample-app/target/*.war)

                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"

                        echo "Uploading file as: $FILE_NAME"

                        curl -u $JFROG_USER:$JFROG_PASS -T "$WAR_FILE" \
                        "https://trial7n02kw.jfrog.io/artifactory/java_warfile_repo-generic-local/$FILE_NAME"
                    '''
                }
            }
        }
    }
}

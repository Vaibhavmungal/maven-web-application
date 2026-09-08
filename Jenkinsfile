pipeline {
    agent any

    environment {
        // Target EC2 instance details for Tomcat 9 deployment
        TOMCAT_IP          = '172.31.28.7' // Replace with your Tomcat EC2 Private or Public IP
        TOMCAT_USER        = 'ec2-user'
        TOMCAT_PATH        = '/opt/apache-tomcat-9.0.73/webapps'
        SSH_CREDENTIAL_ID  = 'ec2-ssh-key' // Credentials ID added in Jenkins
    }

    tools {
        maven 'maven3.9.1'
    }

    options {
        buildDiscarder logRotator(numToKeepStr: '5')
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                // Automatically checks out the current branch and repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building WAR package with Maven..."
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat 9') {
            steps {
                echo "Deploying WAR file to Tomcat on EC2 (${TOMCAT_IP})..."
                sshagent([env.SSH_CREDENTIAL_ID]) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/maven-web-application.war ${TOMCAT_USER}@${TOMCAT_IP}:${TOMCAT_PATH}/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "=========================================================="
            echo " Deployment Succeeded!"
            echo " App URL: http://${TOMCAT_IP}:8080/maven-web-application/"
            echo "=========================================================="
        }
        failure {
            echo "Deployment Failed. Please check the console output above."
        }
    }
}

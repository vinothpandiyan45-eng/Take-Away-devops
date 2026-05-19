pipeline {

    agent {
        label 'slave'
    }

    environment {

        TOMCAT_IP = '172.31.22.237'
        TOMCAT_PATH = '/var/lib/tomcat10/webapps/'

        IMAGE_NAME = 'task-app'
        CONTAINER_NAME = 'task-container'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                url: 'https://github.com/vinothpandiyan45-eng/Take-Away-devops.git'
            }
        }

        stage('Build Maven') {

            steps {

                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Verify Artifact') {

            steps {

                sh 'ls -lh target/'
            }
        }

        stage('Build Docker Image') {

            steps {

                sh """
                docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Stop Old Container') {

            steps {

                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        stage('Run Docker Container') {

            steps {

                sh """
                docker run -d \
                --name ${CONTAINER_NAME} \
                -p 8090:8080 \
                ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Verify Container') {

            steps {

                sh 'docker ps'
            }
        }

        stage('Deploy to Remote Tomcat') {

            steps {

                sshagent(credentials: ['ubuntu']) {

                    sh """
                    scp -o StrictHostKeyChecking=no \
                    target/*.war \
                    ubuntu@${TOMCAT_IP}:${TOMCAT_PATH}myapp.war
                    """
                }
            }
        }
    }

post {
        success {
            mail to: 'vinothpandiyan45@gmail.com',
                 subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Build succeeded: ${env.BUILD_URL}"
        }

        failure {
            mail to: 'vinothpandiyan45@gmail.com',
                 subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Build failed: ${env.BUILD_URL}"
        }
    }
}

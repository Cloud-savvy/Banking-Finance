pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        DOCKER_USER = 'bromaaascripts'
        IMAGE = "${DOCKER_USER}/banking:${BUILD_NUMBER}"
    }

    tools {
        maven 'maven3'  
        jdk 'jdk17'          
    }

    stages {
        stage('Verify Java and Maven') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Git Clone') {
            steps {
                git 'https://github.com/Cloud-savvy/Banking-Finance'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE} ."
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${IMAGE}"
                sh "docker tag ${IMAGE} ${DOCKER_USER}/banking:latest"
                sh "docker push ${DOCKER_USER}/banking:latest"
            }
        }
        stage('Deploy Container') {
            steps {
                sh 'docker run -itd --name banking-container -p 8083:80 ${IMAGE}'
            }
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_USER = "bromaaascripts"
        IMAGE = "${DOCKER_USER}/banking:${BUILD_NUMBER}"
    }

    stages {
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
                sh "docker build -t $IMAGE ."
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
                script {
                    sh "docker push $IMAGE"
                    sh "docker tag $IMAGE ${DOCKER_USER}/banking:latest"
                    sh "docker push ${DOCKER_USER}/banking:latest"
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f banking-container || true
                    docker run -itd --name banking-container -p 8083:8081 $IMAGE
                '''
            }
        }
    }

    post {
        always {
            cleanWs() // clean up workspace only
        }
    }
}

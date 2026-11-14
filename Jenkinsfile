pipeline {
    agent any
    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }
    environment {
        IMAGE_BACK  = "hamed2027/app-gestion-backend"
        IMAGE_FRONT = "hamed2027/app-gestion-frontend"
        MAVEN_OPTS  = "-XX:+UseSerialGC -Xmx512m"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        // ==================== PARTIE BACKEND ====================
        stage('Backend - Build & Test') {
            steps {
                dir('backend') {
                    sh 'mvn clean package'
                }
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }
        stage('Backend - Docker Build & Push') {
            steps {
                dir('backend') {
                    sh "docker build -t ${IMAGE_BACK}:${BUILD_NUMBER} -t ${IMAGE_BACK}:latest ."
                }
                withCredentials([usernamePassword(credentialsId: 'hamed2027', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${IMAGE_BACK}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE_BACK}:latest"
                }
            }
        }
        stage('Backend - Deploy Conteneur') {
            steps {
                sh """
                    docker stop app-backend || true
                    docker rm app-backend || true
                    docker run -d --name app-backend --network host -e SERVER_PORT=8081 ${IMAGE_BACK}:latest
                """
            }
        }

        // ==================== PARTIE FRONTEND ====================
        stage('Frontend - Docker Build & Push') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${IMAGE_FRONT}:${BUILD_NUMBER} -t ${IMAGE_FRONT}:latest ."
                }
                withCredentials([usernamePassword(credentialsId: 'hamed2027', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${IMAGE_FRONT}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE_FRONT}:latest"
                }
            }
        }
        stage('Frontend - Deploy Conteneur') {
            steps {
                sh """
                    docker stop app-frontend || true
                    docker rm app-frontend || true
                    docker run -d --name app-frontend -p 80:80 ${IMAGE_FRONT}:latest
                """
            }
        }
    }
}

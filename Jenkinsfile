pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }
    environment {
        JAVA_HOME    = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH         = "/usr/lib/jvm/java-17-openjdk-amd64/bin:${env.PATH}"
        MAVEN_OPTS   = "-XX:+UseSerialGC -XX:UseAVX=0 -Xms256m -Xmx512m"
        NODE_OPTIONS = "--max-old-space-size=1536"
    }
    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }

        // ==================== PARTIE BACKEND ====================
        stage('2. Backend - Build & Test') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -T 1 -DforkCount=0'
                }
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }
        stage('3. Backend - Deploy') {
            steps {
                sh """
                    pkill -f "app-backend.jar" || true
                    cp backend/target/*.jar /opt/app-backend/app-backend.jar
                    BUILD_ID=dontKillMe nohup /usr/lib/jvm/java-17-openjdk-amd64/bin/java -XX:+UseSerialGC -XX:UseAVX=0 -jar /opt/app-backend/app-backend.jar --server.port=8081 > /opt/app-backend/app.log 2>&1 &
                """
            }
        }

        // ==================== PARTIE FRONTEND ====================
        stage('4. Frontend - Build') {
            steps {
                dir('frontend') {
                    sh 'npm install --legacy-peer-deps'
                    sh 'npm run build'
                }
            }
        }
        stage('5. Frontend - Deploy') {
            steps {
                sh """
                    rm -rf /var/www/app-frontend/*
                    cp -r frontend/dist/frontend/browser/* /var/www/app-frontend/
                """
            }
        }
    }
}

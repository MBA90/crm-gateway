pipeline {
    agent any

    tools {
        // Name must match a JDK 21 installation configured in
        // Manage Jenkins → Tools → JDK installations
        jdk 'jdk-21'
        maven 'maven-3.9'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        // ⚠️ CHANGE THIS to your actual Docker Hub username
        DOCKER_HUB_USER = 'mba90'
        IMAGE_NAME      = 'crm-gateway'
        IMAGE_TAG       = "${BUILD_NUMBER}"

        // This matches the ID you just created in the credentials store
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests -q'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/crm-gateway-*.jar', fingerprint: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Log into Docker Hub
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"

                        // Build the Docker image
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."

                        // Push tag
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
            post {
                always {
                    // Clean up local image to save space on your machine
                    sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
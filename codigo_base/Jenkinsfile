pipeline {
    agent any

    environment {
        IMAGE_TAG = "build-${env.BUILD_NUMBER}"
        NEXUS_REGISTRY   = "localhost:8082"
        NEXUS_MAVEN_REPO = "http://localhost:8081/repository/maven-releases/"
        NEXUS_CREDENTIALS_ID = "nexus-diana"
    }

    stages {
        stage('Checkout & Test') {
            steps {
                checkout scm

                script {
                    echo "The IMAGE TAG is: ${env.IMAGE_TAG}"
                }

                dir('backend') {
                    echo "Compiling and testing backend..."
                    sh 'mvn -B clean verify'
                }
            }
        }

        stage('Package & Tag Inmutable') {
            steps {
                dir('backend') {
                    echo "Packaging backend..."
                    sh 'mvn -B package'
                }
                dir('frontend') {
                    echo "Installing frontend dependencies..."
                    sh 'npm ci'
                    echo "Linting frontend..."
                    sh 'npm run lint'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIALS_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    dir('backend') {
                        sh "mvn deploy -DaltDeploymentRepository=nexus::default::${env.NEXUS_MAVEN_REPO} -Dusername=${NEXUS_USER} -Dpassword=${NEXUS_PASS}"
                    }
                    dir('frontend') {
                        sh "echo \$NEXUS_PASS | docker login ${env.NEXUS_REGISTRY} -u \$NEXUS_USER --password-stdin"
                        sh "docker build -t ${env.NEXUS_REGISTRY}/frontend:${env.IMAGE_TAG} ."
                        sh "docker push ${env.NEXUS_REGISTRY}/frontend:${env.IMAGE_TAG}"
                        sh "docker logout ${env.NEXUS_REGISTRY}"
                    }
                }
            }
        }

        stage('Deploy & Smoke Test') {
            steps {
                sh "echo 'Deploying...'"
            }
        }
    }

    post {
        success {
            echo "Pipeline finalizado en verde. Artefactos publicados con tag: ${env.IMAGE_TAG}"
        }
        failure {
            echo "El pipeline falló. Revise los logs de la etapa correspondiente antes de reintentar."
        }
    }
}

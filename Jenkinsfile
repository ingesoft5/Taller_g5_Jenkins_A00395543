pipeline {
    agent any

    environment {
        IMAGE_TAG = "build-${env.BUILD_NUMBER}"
        NEXUS_REGISTRY   = "nexus:8082"
        NEXUS_MAVEN_REPO = "http://nexus:8081/repository/maven-releases/"
        NEXUS_CREDENTIALS_ID = "nexus-diana"
    }

    stages {
        stage('Checkout & Test') {
            steps {
                checkout scm

                script {
                    echo "The IMAGE TAG is: ${env.IMAGE_TAG}"
                }

                dir('codigo_base/backend') {
                    echo "Compiling and testing backend..."
                    sh 'mvn -B clean verify'
                }
            }
        }

        stage('Package & Tag Inmutable') {
            steps {
                dir('codigo_base/backend') {
                    echo "Packaging backend..."
                    sh 'mvn -B package'
                }
                dir('codigo_base/frontend') {
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
                    dir('codigo_base/backend') {
                        sh '''
                            cat <<EOF > settings.xml
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF
                            mvn deploy -s settings.xml -DaltDeploymentRepository=nexus::default::${NEXUS_MAVEN_REPO}
                            rm -f settings.xml
                        '''
                    }
                    dir('codigo_base/frontend') {
                        sh '''
                            echo "$NEXUS_PASS" | docker login "$NEXUS_REGISTRY" -u "$NEXUS_USER" --password-stdin
                            docker build -t "$NEXUS_REGISTRY/frontend:$IMAGE_TAG" .
                            docker push "$NEXUS_REGISTRY/frontend:$IMAGE_TAG"
                            docker logout "$NEXUS_REGISTRY"
                        '''
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

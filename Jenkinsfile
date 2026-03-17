pipeline {
    agent any

    environment {
        IMAGE_NAME     = "192.168.80.140:80/dev/java-ms-demo"
        IMAGE_TAG      = "1.0"
        SONAR_HOST_URL = "http://192.168.80.140:9000"
        SONAR_LOGIN    = credentials('sonar-token')

        // ✅ BuildKit socket
        BUILDKIT_HOST  = "unix:///run/buildkit/buildkitd.sock"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ravinderrana7173/ms-java.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Scan') {
            steps {
                sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=demo \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_LOGIN
                """
            }
        }

        stage('Build Container Image') {
            steps {
                sh '''
                echo "🔹 Cleaning old images..."
                sudo nerdctl image prune -f || true

                echo "🔹 Building image with BuildKit..."
                sudo -E nerdctl build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-creds',
                    usernameVariable: 'HUSER',
                    passwordVariable: 'HPASS'
                )]) {
                    sh """
                    echo "🔹 Logging into Harbor..."
                    echo \$HPASS | sudo nerdctl login 192.168.80.140:80 -u \$HUSER --password-stdin --insecure-registry

                    echo "🔹 Pushing image..."
                    sudo nerdctl push --insecure-registry $IMAGE_NAME:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        echo "🔹 Deploying to Kubernetes..."
                        kubectl apply -f deployment.yaml --validate=false
                        kubectl apply -f service.yaml
                    """
                }
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs."
        }
    }
}

pipeline {
    agent any

    environment {
        IMAGE_NAME     = "192.168.80.140:80/dev/java-ms-demo"
        IMAGE_TAG      = "1.0"
        SONAR_HOST_URL = "http://192.168.80.140:9000"  // SonarQube server URL
        SONAR_LOGIN    = credentials('sonar-token')   // SonarQube authentication token
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
                // Use the server URL and token explicitly
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
        sudo nerdctl image prune -f || true
        sudo nerdctl build -t $IMAGE_NAME:$IMAGE_TAG .
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
            echo \$HPASS | sudo nerdctl login 192.168.80.140:80 -u \$HUSER --password-stdin --insecure-registry
            sudo nerdctl push --insecure-registry $IMAGE_NAME:$IMAGE_TAG
            """
        }
    }
}

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f deployment.yaml  --validate=false
                        kubectl apply -f service.yaml
                    """
                }
            }
        }

    }
}

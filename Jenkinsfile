pipeline {
    agent any

    environment {
        IMAGE_NAME = "harbor.local/dev/java-ms-demo"
        IMAGE_TAG = "1.0"
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
                sh 'mvn sonar:sonar'
            }
        }

        stage('Build Container Image') {
            steps {
                sh '''
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
                    sh '''
                    echo $HPASS | sudo nerdctl login harbor.local -u $HUSER --password-stdin
                    sudo nerdctl push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }

    }
}


pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {

        stage('Build & Verify Order Service') {
            steps {
                sh '''
                    cd services/order-service
                    mvn clean verify
                '''
            }
        }

        stage('Build & Verify Product Service') {
            steps {
                sh '''
                    cd services/product-service
                    mvn clean verify
                '''
            }
        }
    }

    post {
        success {
            echo 'Both services verified successfully!'
        }

        failure {
            echo 'Build or verification failed!'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

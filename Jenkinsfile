
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ORDER SERVICE

        stage('Build Order Service') {
            steps {
                sh '''
                    cd services/order-service
                    mvn clean compile
                '''
            }
        }

        stage('Test Order Service') {
            steps {
                sh '''
                    cd services/order-service
                    mvn test
                '''
            }
        }

        // PRODUCT SERVICE

        stage('Build Product Service') {
            steps {
                sh '''
                    cd services/product-service
                    mvn clean compile
                '''
            }
        }

        stage('Test Product Service') {
            steps {
                sh '''
                    cd services/product-service
                    mvn test
                '''
            }
        }

        // PACKAGE

        stage('Package Order Service') {
            steps {
                sh '''
                    cd services/order-service
                    mvn package -DskipTests
                '''
            }
        }

        stage('Package Product Service') {
            steps {
                sh '''
                    cd services/product-service
                    mvn package -DskipTests
                '''
            }
        }
    }

    post {
        success {
            echo 'Both microservices built and tested successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}

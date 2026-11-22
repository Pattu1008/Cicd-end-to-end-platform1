
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
        always {

            // Publish JUnit test results for both services
            junit(
                testResults: 'services/*/target/surefire-reports/TEST-*.xml',
                allowEmptyResults: false
            )

            // Archive Order and Product JAR files
            archiveArtifacts(
                artifacts: 'services/*/target/*.jar',
                fingerprint: true
            )

            // Archive JaCoCo HTML, XML and CSV reports
            archiveArtifacts(
                artifacts: 'services/*/target/site/jacoco/**',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'Both services built, tested and verified successfully!'
        }

        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}

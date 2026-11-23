
pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {

        stage('Build & Verify Order Service') {
            steps {
                dir('services/order-service') {
                    sh 'mvn clean verify'
                }
            }
        }

        stage('Build & Verify Product Service') {
            steps {
                dir('services/product-service') {
                    sh 'mvn clean verify'
                }
            }
        }
    }

    post {
        always {

            // Publish JUnit test results for both services
            junit(
                testResults: 'services/*/target/surefire-reports/TEST-*.xml',
                allowEmptyResults: true
            )

            // Publish Order Service coverag
            recordCoverage(
                id: 'order-service',
                name: 'Order Service Coverage',
                tools: [[
                    parser: 'JACOCO',
                    pattern: 'services/order-service/target/site/jacoco/jacoco.xml'
                ]]
            )

            // Publish Product Service coverage
            recordCoverage(
                id: 'product-service',
                name: 'Product Service Coverage',
                tools: [[
                    parser: 'JACOCO',
                    pattern: 'services/product-service/target/site/jacoco/jacoco.xml'
                ]]
            )

            // Archive both service JAR files
            archiveArtifacts(
                artifacts: 'services/*/target/*.jar',
                fingerprint: true,
                allowEmptyArchive: true
            )

            // Archive JaCoCo reports
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

        cleanup {
            echo 'Pipeline execution completed.'
        }
    }
}

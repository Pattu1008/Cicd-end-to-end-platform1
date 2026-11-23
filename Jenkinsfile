
pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {

        // 1. Build and test Order Service
        stage('Build & Verify Order Service') {
            steps {
                dir('services/order-service') {
                    sh 'mvn clean verify'
                }
            }
        }

        // 2. Build and test Product Service
        stage('Build & Verify Product Service') {
            steps {
                dir('services/product-service') {
                    sh 'mvn clean verify'
                }
            }
        }

        // 3. SonarQube analysis for Order Service
        stage('SonarQube Analysis - Order') {
            steps {
                dir('services/order-service') {
                    withSonarQubeEnv('Sonarserver') {
                        withCredentials([
                            string(
                                credentialsId: 'sonar-order-token',
                                variable: 'SONAR_TOKEN'
                            )
                        ]) {
                            sh '''
                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar \
                                  -Dsonar.projectKey=orderservice \
                                  -Dsonar.token=$SONAR_TOKEN \
                                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            '''
                        }
                    }
                }
            }
        }

        // 4. Check Order Service Quality Gate
        stage('Quality Gate - Order') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 5. SonarQube analysis for Product Service
        stage('SonarQube Analysis - Product') {
            steps {
                dir('services/product-service') {
                    withSonarQubeEnv('Sonarserver') {
                        withCredentials([
                            string(
                                credentialsId: 'productservice',
                                variable: 'productservice'
                            )
                        ]) {
                            sh '''
                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar \
                                  -Dsonar.projectKey=Productservice \
                                  -Dsonar.token=$productservice \
                                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            '''
                        }
                    }
                }
            }
        }

        // 6. Check Product Service Quality Gate
        stage('Quality Gate - Product') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {

            // Publish JUnit test results
            junit(
                testResults: 'services/*/target/surefire-reports/TEST-*.xml',
                allowEmptyResults: true
            )

            // Publish Order Service coverage
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
            echo 'Both services built, tested and passed SonarQube Quality Gates!'
        }

        failure {
            echo 'Pipeline failed. Check console output.'
        }

        cleanup {
            echo 'Pipeline execution completed.'
        }
    }
}

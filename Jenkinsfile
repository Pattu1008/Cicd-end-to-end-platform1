pipeline {
    agent {
        label 'agent-1'
    }

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Verify Tools') {
            steps {
                sh '''
                    java -version
                    mvn -version
                '''
            }
        }

        stage('Build & Verify Order Service') {
            steps {
                dir('services/order-service') {
                    sh 'mvn clean verify'
                }
            }
        }

        stage('Build & Verify Product Serice') {
            steps {
                dir('services/product-service') {
                    sh 'mvn clean verify'
                }
            }
        }

        stage('SonarQube Analysis - Order') {
            steps {
                withSonarQubeEnv('Sonarserver') {
                    withCredentials([
                        string(
                            credentialsId: 'sonarordertoken',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {
                        dir('services/order-service') {
                            sh '''
                                set +x

                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar \
                                -Dsonar.projectKey=orderservice \
                                -Dsonar.token="$SONAR_TOKEN" \
                                -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate - Order') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('SonarQube Analysis - Product') {
            steps {
                withSonarQubeEnv('Sonarserver') {
                    withCredentials([
                        string(
                            credentialsId: 'sonarproducttoken',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {
                        dir('services/product-service') {
                            sh '''
                                set +x

                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar \
                                -Dsonar.projectKey=productservice \
                                -Dsonar.token="$SONAR_TOKEN" \
                                -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate - Product') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build docker image for product service') {
            steps {
                dir('services/product-service') {
                    sh 'docker build -t product_service:${BUILD_NUMBER} .'
                }

            }
        }
        stage('Build docker image for order service') {
            steps {
                dir('services/order-service') {
                    sh 'docker build -t order_service:${BUILD_NUMBER} .'
                }

            }
        }
    }

    post {
        always {
            echo 'Publishing test results...'

            junit(
                testResults: 'services/*/target/surefire-reports/TEST-*.xml',
                allowEmptyResults: true
            )

            echo 'Publishing code coverage...'

            recordCoverage(
                tools: [
                    [
                        parser: 'JACOCO',
                        pattern: 'services/order-service/target/site/jacoco/jacoco.xml'
                    ],
                    [
                        parser: 'JACOCO',
                        pattern: 'services/product-service/target/site/jacoco/jacoco.xml'
                    ]
                ],
                enabledForFailure: true
            )

            archiveArtifacts(
                artifacts: 'services/*/target/*.jar,services/*/target/site/jacoco/**',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }

        cleanup {
            echo 'Pipeline execution completed.'
        }
    }
}
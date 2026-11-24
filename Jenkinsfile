
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
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '409953608113'
        ECR_REGISTRY = '409953608113.dkr.ecr.ap-south-1.amazonaws.com'
        PRODUCT_REPO = 'productservice123'
        ORDER_REPO = 'orderservice123'
    }

    stages {

        stage('Verify Tools') {
            steps {
                sh '''
                    java -version
                    mvn -version
                    docker --version
                    aws --version
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

        stage('Build & Verify Product Service') {
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

        stage('Build Docker Image - Product') {
            steps {
                dir('services/product-service') {
                    sh 'docker build -t product_service:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Build Docker Image - Order') {
            steps {
                dir('services/order-service') {
                    sh 'docker build -t order_service:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                    set -e
                    aws ecr get-login-password --region "$AWS_REGION" |
                    docker login --username AWS --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Push Product Image to ECR') {
            steps {
                sh '''
                    set -e

                    docker tag product_service:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${PRODUCT_REPO}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${PRODUCT_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Order Image to ECR') {
            steps {
                sh '''
                    set -e

                    docker tag order_service:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}
                '''
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

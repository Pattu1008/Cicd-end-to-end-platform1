
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

        EKS_CLUSTER_NAME = 'new-cluster'
        K8S_NAMESPACE = 'microservices-cicd'
    }

    stages {

        // 1. Verify Tools
        stage('Verify Tools') {
            steps {
                sh '''
                    set -e

                    echo "Checking Java"
                    java -version

                    echo "Checking Maven"
                    mvn -version

                    echo "Checking Git"
                    git --version

                    echo "Checking Docker"
                    docker --version

                    echo "Checking AWS CLI"
                    aws --version

                    echo "Checking kubectl"
                    kubectl version --client

                    echo "Build Number: $IMAGE_TAG"
                '''
            }
        }

        // 2. Parallel Build and Test
        stage('Parallel Build and Test') {
            parallel {

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
        }

        // 3. SonarQube Analysis - Order
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
                                set -e
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

        // 4. Quality Gate - Order
        stage('Quality Gate - Order') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 5. SonarQube Analysis - Product
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
                                set -e
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

        // 6. Quality Gate - Product
        stage('Quality Gate - Product') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 7. Build Product Docker Image
        stage('Build Docker Image - Product') {
            steps {
                dir('services/product-service') {
                    sh '''
                        set -e

                        docker build \
                        -t product-service:${IMAGE_TAG} .
                    '''
                }
            }
        }

        // 8. Build Order Docker Image
        stage('Build Docker Image - Order') {
            steps {
                dir('services/order-service') {
                    sh '''
                        set -e

                        docker build \
                        -t order-service:${IMAGE_TAG} .
                    '''
                }
            }
        }

        // 9. Login to AWS ECR
        stage('Login to AWS ECR') {
            steps {
                sh '''
                    set -e

                    aws ecr get-login-password \
                    --region "$AWS_REGION" |
                    docker login \
                    --username AWS \
                    --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        // 10. Push Product Image
        stage('Push Product Image to ECR') {
            steps {
                sh '''
                    set -e

                    docker tag \
                    product-service:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${PRODUCT_REPO}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${PRODUCT_REPO}:${IMAGE_TAG}
                '''
            }
        }

        // 11. Push Order Image
        stage('Push Order Image to ECR') {
            steps {
                sh '''
                    set -e

                    docker tag \
                    order-service:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}
                '''
            }
        }

        // 12. Verify ECR Images
        stage('Verify ECR Images') {
            steps {
                sh '''
                    set -e

                    echo "Verifying Product Image"

                    aws ecr describe-images \
                    --repository-name "$PRODUCT_REPO" \
                    --image-ids imageTag="$IMAGE_TAG" \
                    --region "$AWS_REGION"

                    echo "Verifying Order Image"

                    aws ecr describe-images \
                    --repository-name "$ORDER_REPO" \
                    --image-ids imageTag="$IMAGE_TAG" \
                    --region "$AWS_REGION"
                '''
            }
        }

        // 13. Configure EKS Connection
        stage('Configure EKS Connection') {
            steps {
                sh '''
                    set -e

                    aws eks update-kubeconfig \
                    --region "$AWS_REGION" \
                    --name "$EKS_CLUSTER_NAME"

                    kubectl get nodes
                '''
            }
        }

        // 14. Deploy Kubernetes Configurations
        stage('Deploy Kubernetes Configurations') {
            steps {
                sh '''
                    set -e

                    kubectl apply -f k8s/namespace.yaml

                    kubectl apply -f k8s/limit-range.yaml
                    kubectl apply -f k8s/resource-quota.yaml
                    kubectl apply -f k8s/serviceaccount.yaml

                    kubectl apply -f k8s/product-configmap.yaml
                    kubectl apply -f k8s/product-secret.yaml

                    kubectl apply -f k8s/order-configmap.yaml
                '''
            }
        }

        // 15. Deploy Product Service
        stage('Deploy Product Service to EKS') {
            steps {
                sh '''
                    set -e

                    echo "Deploying Product Service"
                    echo "Image Tag: $IMAGE_TAG"

                    sed "s|IMAGE_TAG_PLACEHOLDER|${IMAGE_TAG}|g" \
                    k8s/product-deployment.yaml > /tmp/product-deployment.yaml

                    if grep -q "IMAGE_TAG_PLACEHOLDER" /tmp/product-deployment.yaml; then
                        echo "ERROR: Product image placeholder was not replaced"
                        exit 1
                    fi

                    kubectl apply -f /tmp/product-deployment.yaml
                    kubectl apply -f k8s/product-service.yaml

                    kubectl rollout status \
                    deployment/product-deployment \
                    -n "$K8S_NAMESPACE" \
                    --timeout=180s
                '''
            }
        }

        // 16. Deploy Order Service
        stage('Deploy Order Service to EKS') {
            steps {
                sh '''
                    set -e

                    echo "Deploying Order Service"
                    echo "Image Tag: $IMAGE_TAG"

                    sed "s|IMAGE_TAG_PLACEHOLDER|${IMAGE_TAG}|g" \
                    k8s/order-deployment.yaml > /tmp/order-deployment.yaml

                    if grep -q "IMAGE_TAG_PLACEHOLDER" /tmp/order-deployment.yaml; then
                        echo "ERROR: Order image placeholder was not replaced"
                        exit 1
                    fi

                    kubectl apply -f /tmp/order-deployment.yaml
                    kubectl apply -f k8s/order-service.yaml

                    kubectl rollout status \
                    deployment/order-deployment \
                    -n "$K8S_NAMESPACE" \
                    --timeout=180s
                '''
            }
        }

        // 17. Verify Kubernetes Deployment
        stage('Verify Kubernetes Deployment') {
            steps {
                sh '''
                    set -e

                    echo "Checking Deployments"
                    kubectl get deployments -n "$K8S_NAMESPACE"

                    echo "Checking Pods"
                    kubectl get pods -n "$K8S_NAMESPACE" -o wide

                    echo "Checking Services"
                    kubectl get services -n "$K8S_NAMESPACE"

                    echo "Checking HPA"
                    kubectl get hpa -n "$K8S_NAMESPACE"

                    echo "Checking Resource Quota"
                    kubectl get resourcequota -n "$K8S_NAMESPACE"

                    echo "Checking Deployed Images"
                    kubectl get deployments \
                    -n "$K8S_NAMESPACE" \
                    -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.spec.template.spec.containers[*].image}{"\\n"}{end}'
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
            echo "Deployed Image Tag: ${IMAGE_TAG}"
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }

        cleanup {
            echo 'Pipeline execution completed.'
        }
    }
}


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

        HELM_RELEASE = 'microservices1'
        HELM_CHART = './microservices1'
    }

    stages {

        stage('Verify Tools') {
            steps {
                sh '''
                    set -e
                    java -version
                    mvn -version
                    git --version
                    docker --version
                    aws --version
                    kubectl version --client
                    helm version

                    echo "Build Number: $IMAGE_TAG"
                '''
            }
        }

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

        stage('SonarQube Analysis - Order') {
            steps {
                withSonarQubeEnv('Sonarserver') {
                    withCredentials([
                        string(credentialsId: 'sonarordertoken',
                               variable: 'SONAR_TOKEN')
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
                        string(credentialsId: 'sonarproducttoken',
                               variable: 'SONAR_TOKEN')
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
                    sh '''
                        set -e
                        docker build -t product-service:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Build Docker Image - Order') {
            steps {
                dir('services/order-service') {
                    sh '''
                        set -e
                        docker build -t order-service:${IMAGE_TAG} .
                    '''
                }
            }
        }

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

        stage('Push Product Image to ECR') {
            steps {
                sh '''
                    set -e

                    docker tag product-service:${IMAGE_TAG} \
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

                    docker tag order-service:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${ORDER_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify ECR Images') {
            steps {
                sh '''
                    set -e

                    aws ecr describe-images \
                    --repository-name "$PRODUCT_REPO" \
                    --image-ids imageTag="$IMAGE_TAG" \
                    --region "$AWS_REGION"

                    aws ecr describe-images \
                    --repository-name "$ORDER_REPO" \
                    --image-ids imageTag="$IMAGE_TAG" \
                    --region "$AWS_REGION"
                '''
            }
        }

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

        stage('Validate Helm Chart') {
            steps {
                sh '''
                    set -e

                    echo "Checking Helm chart"

                    helm lint "$HELM_CHART"

                    echo "Rendering Helm templates"

                    helm template "$HELM_RELEASE" "$HELM_CHART" \
                    --namespace "$K8S_NAMESPACE" \
                    --set product.image.tag="$IMAGE_TAG" \
                    --set order.image.tag="$IMAGE_TAG" \
                    > rendered.yaml

                    echo "Helm rendering completed"
                '''
            }
        }

        stage('Deploy Using Helm') {
            steps {
                sh '''
                    set -e

                    echo "Deploying Product and Order services"
                    echo "Image Tag: $IMAGE_TAG"

                    helm upgrade --install "$HELM_RELEASE" "$HELM_CHART" \
                    --namespace "$K8S_NAMESPACE" \
                    --set product.image.tag="$IMAGE_TAG" \
                    --set order.image.tag="$IMAGE_TAG" \
                    --wait \
                    --timeout 5m

                    echo "Helm deployment completed"
                '''
            }
        }

        stage('Verify Helm Deployment') {
            steps {
                sh '''
                    set -e

                    echo "Helm Release Status"
                    helm status "$HELM_RELEASE" \
                    --namespace "$K8S_NAMESPACE"

                    echo "Deployments"
                    kubectl get deployments -n "$K8S_NAMESPACE"

                    echo "Pods"
                    kubectl get pods -n "$K8S_NAMESPACE" -o wide

                    echo "Services"
                    kubectl get services -n "$K8S_NAMESPACE"

                    echo "HPA"
                    kubectl get hpa -n "$K8S_NAMESPACE"

                    echo "Deployed Images"
                    kubectl get deployments \
                    -n "$K8S_NAMESPACE" \
                    -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.spec.template.spec.containers[*].image}{"\n"}{end}'
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
                artifacts: 'services/*/target/*.jar,services/*/target/site/jacoco/**,rendered.yaml',
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

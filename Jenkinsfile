pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/jenkins_home/jenkins-kubeconfig'
    }

    stages {

        stage('Start') {
            steps {
                echo '=============================================='
                echo 'CLOUD NATIVE DEVOPS AUTOMATION SYSTEM'
                echo '=============================================='
                echo "Application: ScholarOS"
                echo "Environment: ${params.ENVIRONMENT}"
                echo "Application URL: ${params.APPLICATION_URL}"
                echo 'Pipeline started successfully.'
            }
        }

        stage('Backend Health Check') {
            when {
                expression {
                    return params.RUN_TESTS
                }
            }
            steps {
                sh '''
                    echo "Checking ScholarOS backend..."

                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://172.17.0.1:5478/api/health)

                    echo "Backend HTTP Status: $STATUS"

                    if [ "$STATUS" = "200" ]; then
                        echo "✓ ScholarOS backend is healthy"
                    else
                        echo "✗ ScholarOS backend health check failed"
                        exit 1
                    fi
                '''
            }
        }

        stage('Frontend Build') {
            when {
                expression {
                    return params.RUN_TESTS
                }
            }
            steps {
                dir('frontend') {
                    sh '''
                        echo "Installing frontend dependencies..."
                        npm ci

                        echo "Building ScholarOS frontend..."
                        npm run build

                        echo "✓ Frontend build completed successfully"
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "=============================================="
                    echo "DOCKER BUILD"
                    echo "=============================================="

                    echo "Building ScholarOS Docker image..."

                    docker build -t scholaros:${BUILD_NUMBER} .

                    echo "Finding Minikube network address..."

                    REGISTRY_PORT=$(docker port minikube 5000/tcp | sed 's/.*://')

                    if [ -z "$REGISTRY_PORT" ]; then
                        echo "✗ Could not determine Minikube registry port"
                        exit 1
                    fi

                    REGISTRY="127.0.0.1:${REGISTRY_PORT}"

                    echo "Minikube registry port: $REGISTRY_PORT"
                    echo "Registry: $REGISTRY"

                    echo "Tagging Docker image..."

                    docker tag \
                        scholaros:${BUILD_NUMBER} \
                        ${REGISTRY}/scholaros:${BUILD_NUMBER}

                    echo "Pushing image to Minikube registry..."

                    docker push \
                        ${REGISTRY}/scholaros:${BUILD_NUMBER}

                    echo "✓ Docker image built and pushed successfully"
                '''
            }
        }

        stage('Kubernetes Deployment') {
            when {
                expression {
                    return params.DEPLOY_TO_K8S
                }
            }
            steps {
                sh '''
                    echo "=============================================="
                    echo "KUBERNETES DEPLOYMENT"
                    echo "=============================================="

                    echo "Checking Kubernetes connection..."

                    kubectl get namespace scholaros

                    echo "Applying ScholarOS Kubernetes configuration..."

                    kubectl apply -f k8s/scholaros.yaml

                    echo "Updating application image..."

                    kubectl -n scholaros set image \
                        deployment/scholaros-app \
                        scholaros=registry.minikube/scholaros:${BUILD_NUMBER}

                    echo "Waiting for deployment rollout..."

                    kubectl -n scholaros rollout status \
                        deployment/scholaros-app \
                        --timeout=180s

                    echo "----------------------------------------------"
                    echo "Kubernetes deployment status:"
                    echo "----------------------------------------------"

                    kubectl -n scholaros get deployment scholaros-app
                    kubectl -n scholaros get pods -l app=scholaros-app

                    echo "✓ ScholarOS deployment completed successfully"
                '''
            }
        }

        stage('Application Verification') {
            when {
                expression {
                    return params.DEPLOY_TO_K8S
                }
            }
            steps {
                sh '''
                    echo "=============================================="
                    echo "APPLICATION VERIFICATION"
                    echo "=============================================="

                    echo "Checking ScholarOS Kubernetes service..."

                    kubectl --kubeconfig=/var/jenkins_home/jenkins-kubeconfig -n scholaros port-forward \
                        svc/scholaros-service 5478:5478 \
                        > /tmp/scholaros-port-forward.log 2>&1 &

                    PF_PID=$!

                    trap 'kill $PF_PID 2>/dev/null || true' EXIT

                    echo "Waiting for ScholarOS service..."

                    for i in $(seq 1 20); do
                        STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
                            http://127.0.0.1:5478/api/health || true)

                        if [ "$STATUS" = "200" ]; then
                            break
                        fi

                        sleep 2
                    done

                    echo "Application HTTP Status: $STATUS"

                    if [ "$STATUS" = "${EXPECTED_STATUS}" ]; then
                        echo "✓ ScholarOS application is healthy"
                        echo "✓ Kubernetes service verification SUCCESS"
                    else
                        echo "✗ ScholarOS application health verification failed"
                        echo "Port-forward output:"
                        cat /tmp/scholaros-port-forward.log
                        exit 1
                    fi
                 '''
            }
        }
    }

    post {
        success {
            echo '=============================================='
            echo 'BUILD SUCCESS'
            echo '=============================================='
            echo 'ScholarOS CI/CD pipeline completed successfully.'
        }

        failure {
            echo '=============================================='
            echo 'BUILD FAILURE'
            echo '=============================================='
            echo 'ScholarOS CI/CD pipeline failed.'
            echo 'Check the failed stage and console output.'
        }
    }
}

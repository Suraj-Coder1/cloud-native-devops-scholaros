pipeline {
        environment {
        KUBECONFIG = '/var/jenkins_home/jenkins-kubeconfig'
    }
    agent any

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
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }
        stage('Docker Build') {
    steps {
        sh '''
            echo "Building ScholarOS Docker image..."

            docker build -t scholaros:${BUILD_NUMBER} .

            docker tag scholaros:${BUILD_NUMBER} 127.0.0.1:32770/scholaros:${BUILD_NUMBER}

            echo "Pushing image to Minikube registry..."

            docker push 127.0.0.1:32770/scholaros:${BUILD_NUMBER}

            echo "Docker image built and pushed successfully"
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
                    echo "Deploying ScholarOS to Kubernetes..."

                    kubectl -n scholaros set image \
                      deployment/scholaros-app \
                      scholaros=registry.kube-system.svc.cluster.local/scholaros:${BUILD_NUMBER}

                    echo "Waiting for deployment rollout..."

                    kubectl -n scholaros rollout status \
                      deployment/scholaros-app \
                      --timeout=180s

                    echo "ScholarOS deployment completed successfully."
                '''
            }
        }
    }
}


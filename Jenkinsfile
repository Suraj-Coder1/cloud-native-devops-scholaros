pipeline {
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
    }
}

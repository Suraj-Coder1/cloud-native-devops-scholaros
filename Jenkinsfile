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

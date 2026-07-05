pipeline {
    agent any

    environment {
        // Define your staging deployment path or server destination
        STAGING_DIR = '/var/www/flask_staging'
    }

    triggers {
        // Triggers a build when GitHub sends a push webhook
        githubPush()
    }

    stages {
        stage('Build') {
            steps {
                echo 'Setting up Python virtual environment and installing dependencies...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running Unit Tests with pytest...'
                sh '''
                    . venv/bin/activate
                    pytest --junitxml=reports/results.xml
                '''
            }
            post {
                always {
                    // Captures and displays test results in the Jenkins UI
                    junit 'reports/results.xml'
                }
            }
        }

        stage('Deploy to Staging') {
            // Only deploys if changes are pushed to the main branch
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying application to Staging environment...'
                // Example deployment using rsync or a local service restart script
                sh '''
                    . venv/bin/activate
                    mkdir -p ${STAGING_DIR}
                    rsync -avz --exclude='venv' ./ ${STAGING_DIR}
                    # Example command to reload your WSGI server (e.g., gunicorn)
                    # sudo systemctl restart flask_staging.service
                '''
            }
        }
    }

    post {
        success {
            emailNotification('SUCCESS', 'The build completed successfully.')
        }
        failure {
            emailNotification('FAILURE', 'The build failed. Please check the console output.')
        }
    }
}

// Helper function to keep notifications clean and reusable
def emailNotification(String status, String message) {
    mail to: 'your-email@example.com',
         subject: "Jenkins Build ${status}: ${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]",
         body: "${message}\n\nView the logs here: ${env.BUILD_URL}"
}
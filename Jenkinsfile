pipeline {
    agent any

    environment {
        FIREBASE_TOKEN = credentials('firebase-token')
        FIREBASE_PROJECT = 'khoiltn-workshop2'
        // ANSIBLE_VENV = '/ansible_venv'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                echo 'Dependencies installed successfully'
            }
        }

        stage('Lint/Test') {
            steps {
                sh 'npm run test:ci'
                echo 'Lint and tests passed successfully'
            }
        }

        stage('Deploy to Firebase') {
            steps {
                script {
                    try {
                        sh '''
                            firebase deploy --token "$FIREBASE_TOKEN" --only hosting --project="$FIREBASE_PROJECT"
                        '''
                        echo 'Firebase deployment successful'
                    } catch (Exception e) {
                        echo "Firebase deployment failed: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend (
                color: 'good',
                message: "✅ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thành công!"
            )
        }
        failure {
            slackSend (
                color: 'danger',
                message: "❌ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thất bại!"
            )
        }
    }
}

pipeline {
    agent any
    
    environment {
        // FIREBASE_TOKEN = credentials('firebase-token')
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
        
        // stage('Deploy to Firebase') {
        //     steps {
        //         script {
        //             try {
        //                 sh '''
        //                     export GOOGLE_APPLICATION_CREDENTIALS=/tmp/firebase-credentials.json
        //                     echo "$FIREBASE_TOKEN" > /tmp/firebase-credentials.json
        //                     firebase deploy --token "$FIREBASE_TOKEN" --only hosting --project="$FIREBASE_PROJECT"
        //                 '''
        //                 echo 'Firebase deployment successful'
        //             } catch (Exception e) {
        //                 echo "Firebase deployment failed: ${e.getMessage()}"
        //                 throw e
        //             }
        //         }
        //     }
        // }
        
        // stage('Deploy to Remote with Ansible') {
        //     steps {
        //         script {
        //             try {
        //                 sh '''
        //                     # Activate Ansible virtual environment
        //                     source $ANSIBLE_VENV/bin/activate
                            
        //                     # Copy SSH key to Jenkins home
        //                     mkdir -p /var/jenkins_home/.ssh
        //                     cp /var/jenkins_home/ansible/newbie_id_rsa /var/jenkins_home/.ssh/
        //                     chmod 600 /var/jenkins_home/.ssh/newbie_id_rsa
                            
        //                     # Copy source files to Ansible workspace
        //                     cp -r index.html 404.html css/ js/ images/ /var/jenkins_home/ansible/
                            
        //                     # Run Ansible playbook
        //                     cd /var/jenkins_home/ansible
        //                     ansible-playbook -i hosts deploy.yml
        //                 '''
        //                 echo 'Remote deployment with Ansible successful'
        //             } catch (Exception e) {
        //                 echo "Remote deployment failed: ${e.getMessage()}"
        //                 throw e
        //             }
        //         }
        //     }
        // }
    }
    
    post {
        success {
            slackSend (
                channel: '#lnd-2025-workshop',
                color: 'good',
                message: "✅ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thành công! ��"
            )
        }
        failure {
            slackSend (
                channel: '#lnd-2025-workshop',
                color: 'danger',
                message: "❌ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thất bại! 💥"
            )
        }
    }
}

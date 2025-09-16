pipeline {
    agent any

    environment {
        FIREBASE_TOKEN = credentials('firebase-token')
        FIREBASE_PROJECT = 'khoiltn-workshop2'
        REMOTE_HOST = '118.69.34.46'
        REMOTE_PORT = '3334'
        REMOTE_USER = 'newbie'
        REMOTE_KEY = credentials('remote-ssh-key')
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

        stage('Deploy to Local') {
            steps {
                script {
                    try {
                        sh '''
                            # Create local deployment directory structure
                            mkdir -p /var/jenkins_home/khoiltn2/web-performance-project1-initial
                            mkdir -p /var/jenkins_home/khoiltn2/deploy

                            # Copy files to local deployment
                            cp -r index.html 404.html css/ js/ images/ /var/jenkins_home/khoiltn2/web-performance-project1-initial/

                            # Create timestamped deployment
                            TIMESTAMP=$(date +%Y%m%d%H%M%S)
                            cp -r /var/jenkins_home/khoiltn2/web-performance-project1-initial /var/jenkins_home/khoiltn2/deploy/$TIMESTAMP

                            # Update symlink
                            cd /var/jenkins_home/khoiltn2/deploy
                            rm -f current
                            ln -s $TIMESTAMP current

                            # Keep only 5 latest deployments
                            ls -t | tail -n +6 | xargs -r rm -rf
                        '''
                        echo 'Local deployment successful'
                    } catch (Exception e) {
                        echo "Local deployment failed: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }

        stage('Deploy to Remote with SSH') {
            steps {
                script {
                    try {
                        // Sử dụng Jenkins credentials: remote_ssh_key
                        withCredentials([sshUserPrivateKey(credentialsId: 'remote-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                            sh '''
                                REMOTE_DIR="/usr/share/nginx/html/jenkins/khoiltn2"
                                PROJECT_DIR="$REMOTE_DIR/web-performance-project1-initial"
                                DEPLOY_DIR="$REMOTE_DIR/deploy"

                                # Tạo cấu trúc thư mục trên server
                                ssh -i $SSH_KEY -o StrictHostKeyChecking=no -p $REMOTE_PORT $SSH_USER@$REMOTE_HOST "
                                    mkdir -p $PROJECT_DIR
                                    mkdir -p $DEPLOY_DIR
                                "

                                # Copy source code lên server
                                scp -i $SSH_KEY -o StrictHostKeyChecking=no -P $REMOTE_PORT -r index.html 404.html css/ js/ images/ $SSH_USER@$REMOTE_HOST:$PROJECT_DIR/

                                # Tạo bản release timestamp
                                TIMESTAMP=$(date +%Y%m%d%H%M%S)
                                ssh -i $SSH_KEY -o StrictHostKeyChecking=no -p $REMOTE_PORT $SSH_USER@$REMOTE_HOST "
                                    cp -r $PROJECT_DIR $DEPLOY_DIR/$TIMESTAMP
                                    cd $DEPLOY_DIR
                                    rm -f current
                                    ln -s $TIMESTAMP current
                                    ls -t | tail -n +6 | xargs -r rm -rf
                                    echo 'Remote deployment completed: $TIMESTAMP'
                                "
                            '''
                        }
                        echo 'Remote deployment with SSH successful'
                    } catch (Exception e) {
                        echo "Remote deployment failed: ${e.getMessage()}"
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
                message: "✅ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thành công! 🚀"
            )
        }
        failure {
            slackSend (
                color: 'danger',
                message: "❌ khoiltn deploy job ${env.JOB_NAME} #${env.BUILD_NUMBER} thất bại! 💥"
            )
        }
    }
}

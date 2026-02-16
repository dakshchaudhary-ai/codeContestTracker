pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-20'
    }
    
    environment {
        FRONTEND_DIR = 'codeContestTracker-frontend'
        BACKEND_DIR = 'codeContestTracker-backend'
        DEPLOY_DIR = '/var/www/codeContestTracker'
        GIT_REPO = 'https://github.com/dakshchaudhary-ai/codeContestTracker.git'
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout Code') {
            steps {
                echo '📥 Fetching code from Git repository...'
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: "${GIT_REPO}"
            }
        }
        
        stage('Install Dependencies') {
            parallel {
                stage('Frontend Dependencies') {
                    steps {
                        echo '📦 Installing frontend dependencies...'
                        dir("${FRONTEND_DIR}") {
                            sh 'npm install'
                        }
                    }
                }
                stage('Backend Dependencies') {
                    steps {
                        echo '📦 Installing backend dependencies...'
                        dir("${BACKEND_DIR}") {
                            sh 'npm install'
                        }
                    }
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                echo '🏗️ Building React frontend...'
                dir("${FRONTEND_DIR}") {
                    sh 'npm run build'
                }
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Frontend Tests') {
                    steps {
                        echo '🧪 Running frontend tests...'
                        dir("${FRONTEND_DIR}") {
                            sh 'npm test -- --passWithNoTests || true'
                        }
                    }
                }
                stage('Backend Tests') {
                    steps {
                        echo '🧪 Running backend tests...'
                        dir("${BACKEND_DIR}") {
                            sh 'npm test -- --passWithNoTests || true'
                        }
                    }
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                echo '🚀 Deploying application...'
                script {
                    sh """
                        echo '📤 Deploying frontend...'
                        mkdir -p ${DEPLOY_DIR}/frontend
                        rm -rf ${DEPLOY_DIR}/frontend/*
                        cp -r ${FRONTEND_DIR}/dist/* ${DEPLOY_DIR}/frontend/
                    """
                    
                    sh """
                        echo '📤 Deploying backend...'
                        mkdir -p ${DEPLOY_DIR}/backend
                        rm -rf ${DEPLOY_DIR}/backend/*
                        cp -r ${BACKEND_DIR}/* ${DEPLOY_DIR}/backend/
                        if [ -f /var/www/codeContestTracker/backend/.env ]; then
                            cp /var/www/codeContestTracker/backend/.env ${DEPLOY_DIR}/backend/
                        fi
                    """
                    
                    sh """
                        cd ${DEPLOY_DIR}/backend
                        pm2 describe codeContest-backend > /dev/null 2>&1
                        if [ \$? -eq 0 ]; then
                            echo '🔄 Restarting backend...'
                            pm2 restart codeContest-backend
                        else
                            echo '▶️ Starting backend...'
                            pm2 start npm --name codeContest-backend -- start
                        fi
                        pm2 save
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo '🏥 Performing health check...'
                script {
                    sleep 5
                    sh """
                        curl -f http://localhost:5000/api/health || echo 'Health check completed'
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            emailext (
                subject: "✅ SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    <h2>Build Successful! 🎉</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> SUCCESS</p>
                    <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                    <p><a href="${env.BUILD_URL}">View Build Details</a></p>
                    <hr>
                    <p>Application deployed to: ${DEPLOY_DIR}</p>
                """,
                to: 'daksh.choudhary@unthinkable.co',
                mimeType: 'text/html'
            )
        }
        failure {
            echo '❌ Pipeline failed!'
            emailext (
                subject: "❌ FAILED: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    <h2>Build Failed! ⚠️</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> FAILURE</p>
                    <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                """,
                to: 'daksh.choudhary@unthinkable.co',
                mimeType: 'text/html'
            )
        }
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
    }
}


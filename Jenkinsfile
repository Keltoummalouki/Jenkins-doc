pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning Repository...'
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                echo 'Checking environment...'
                sh 'pwd && ls -la'
            }
        }
        
        stage('Mock Install Dependencies') {
            steps {
                echo 'Would install dependencies with: npm ci'
                echo 'Simulating dependency installation...'
            }
        }
        
        stage('Mock Lint') {
            steps {
                echo 'Would run linting with: npm run lint'
                echo 'Simulating code linting...'
            }
        }

        stage('Mock Test') {
            steps {
                echo 'Would run tests with: npm run test'
                echo 'Simulating test execution...'
            }
        }
        
        stage('Mock Build') {
            steps {
                echo 'Would build with: npm run build'
                echo 'Simulating build process...'
                sh 'mkdir -p dist && echo "Build output" > dist/index.html'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/bravo1goingdark/Jenkins-Pipeline-for-a-Static-Website.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building static site...'
                sh 'mkdir -p build && cp -r * build/'
            }
        }

        stage('Deploy to S3') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                    s3Upload(
                        bucket: 's3-jenkins-static',
                        includePathPattern: '**/*',
                        workingDir: 'build'
                    )
                }
            }
        }
    }

    post {
        success { echo '✅ Deployment successful!' }
        failure { echo '❌ Deployment failed.' }
    }
}

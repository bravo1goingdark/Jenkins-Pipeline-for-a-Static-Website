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
                sh '''
                    rm -rf build
                    mkdir build
                    for item in *; do
                        [ "$item" = "build" ] && continue
                        cp -r "$item" build/
                    done
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying static site...'
                sh 'aws s3 sync build/ s3://your-bucket-name --delete'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}


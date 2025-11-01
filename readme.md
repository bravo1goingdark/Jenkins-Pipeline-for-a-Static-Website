# Jenkins + GitHub + AWS S3 Static Website Pipeline

This repository contains a Jenkins pipeline that automatically deploys a static website to an AWS S3 bucket whenever you push code to GitHub. Jenkins runs on your local machine and receives GitHub webhook events through an ngrok tunnel.

---

## Prerequisites

- Jenkins server (local installation or Docker container)
- GitHub repository containing your static site
- AWS account with an S3 bucket configured for static website hosting
- ngrok (or another tunneling tool) to expose Jenkins to GitHub
- AWS CLI (optional, useful for local testing)
- Jenkins plugins:
  - GitHub Plugin
  - GitHub Integration Plugin
  - AWS Steps Plugin

---

## Pipeline Flow

1. Developer pushes changes to the `main` branch.
2. GitHub fires a webhook to the ngrok HTTPS endpoint.
3. ngrok forwards the request to Jenkins running on `http://localhost:8080`.
4. Jenkins executes the pipeline:
   - **Checkout**: clones the repository using the `github-token` credentials.
   - **Build**: copies the workspace into a fresh `build` directory.
   - **Deploy to S3**: uploads the build directory to the `s3-jenkins-static` bucket with the `aws-creds` credentials.
5. The updated static website is served from `http://s3-jenkins-static.s3-website-ap-south-1.amazonaws.com`.

---

## Architecture Overview

```
+------------------+        Push        +--------------------+
|     Developer    | -----------------> |  GitHub Repository |
+------------------+                    +--------------------+
               |                                   |
               | Webhook (push event)              |
               v                                   v
        +------------------+   HTTPS tunnel   +----------------+
        |       ngrok      | ----------------> |    Jenkins     |
        +------------------+                   +----------------+
                                                     |
                                                     | S3 upload
                                                     v
                                              +----------------+
                                              |  AWS S3 Bucket |
                                              +----------------+
```

---

## Jenkinsfile Highlights

```groovy
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
                    find . -maxdepth 1 ! -name build ! -name . -exec cp -r {} build/ \\;
                '''
            }
        }

        stage('Deploy to S3') {
            steps {
                echo 'Uploading to S3...'
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
        success { echo 'Deployment successful!' }
        failure { echo 'Deployment failed.' }
    }
}
```

---

## Setup Guide

1. **AWS**
   - Create the S3 bucket `s3-jenkins-static`.
   - Enable static website hosting.
   - Make the bucket public or configure an IAM policy that allows public reads.

2. **Jenkins**
   - Install the required plugins listed above.
   - Add credentials:
     - Secret text credential with ID `github-token` containing your GitHub personal access token.
     - AWS credential with ID `aws-creds` containing your access key ID and secret access key.
   - Create a Pipeline job named `static-site-deploy` and select "Pipeline script from SCM."
   - Point the job to this GitHub repository and use the `main` branch.

3. **Expose Jenkins**
   - Run `ngrok http 8080`.
   - Copy the HTTPS forwarding URL shown in the ngrok console.

4. **Configure the GitHub Webhook**
   - In your GitHub repository, open **Settings > Webhooks > Add webhook**.
   - Payload URL: `https://<ngrok-id>.ngrok-free.app/github-webhook/`
   - Content type: `application/json`
   - Events: "Just the push event"

---

## Testing the Pipeline

1. Edit a file such as `index.html`.
2. Commit and push your changes to the `main` branch.
3. Jenkins should start the pipeline automatically (look for "Started by GitHub push" in the build log).
4. After the deployment succeeds, reload the S3 static website URL and confirm the new content.

---

## Troubleshooting

| Symptom | Likely Cause | Suggested Fix |
| --- | --- | --- |
| Webhook returns 403 | Jenkins CSRF protection is blocking the request | Temporarily disable "Prevent Cross Site Request Forgery exploits" in Jenkins or configure the GitHub hook correctly |
| Webhook returns 405 | Missing GitHub plugins | Install both "GitHub Plugin" and "GitHub Integration Plugin" |
| Credential ID not found | Jenkins credentials not configured | Create credentials with IDs `github-token` and `aws-creds` under **Manage Jenkins > Credentials** |
| Files not updating on S3 | Wrong bucket, region, or IAM policy | Verify the bucket name, region, and the permissions associated with the AWS credentials |

---

## Future Enhancements

- Front the S3 bucket with CloudFront for HTTPS and caching.
- Add automated tests or linting before the deploy stage.
- Run the pipeline inside a Docker agent for reproducible builds.
- Send Slack or email notifications when the pipeline finishes.

---

## Author

Ashutosh Kumar ([@bravo1goingdark](https://github.com/bravo1goingdark))


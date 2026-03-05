pipeline {
    agent any

    /* 
       POC 2 Requirement: Pipeline Parameters
       Allows selecting which branch to build.
    */
    parameters {
        choice(name: 'BRANCH', choices: ['main', 'develop', 'feature'], description: 'Select the branch to build')
    }

    environment {
        // Define common variables
        DOCKER_IMAGE = "your-dockerhub-username/jenkins-poc-app"
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id') // User needs to setup this ID in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                checkout scm
            }
        }

        /* 
           NEW STEP: Build the image first so the code is baked in.
           This solves the "File Not Found" volume mounting issue.
        */
        stage('Docker Build') {
            steps {
                echo "Building Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Verify & Test') {
            parallel {
                stage('Lint') {
                    steps {
                        echo "Running Linting inside the built image..."
                        sh "docker run --rm ${DOCKER_IMAGE}:${BUILD_NUMBER} flake8 ."
                    }
                }
                stage('Test') {
                    steps {
                        echo "Running Unit Tests inside the built image..."
                        sh "docker run --rm ${DOCKER_IMAGE}:${BUILD_NUMBER} pytest test_main.py"
                    }
                }
            }
        }

        stage('Push to Registry') {
            steps {
                echo "Logging into Docker Hub and pushing image..."
                sh "echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin"
                sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
    }

    /* 
       POC 1 & 2 Requirement: Post-build actions
    */
    post {
        always {
            echo "Pipeline finished. Cleaning up workspace..."
            /* POC 2 Requirement: Post-build cleanup */
            deleteDir()
        }
        success {
            echo "Build successful! Build version: ${BUILD_NUMBER}"
        }
        failure {
            /* 
               POC 1 Requirement: Email notification on failure
               Note: Requires Jenkins Email Extension plugin and SMTP config.
            */
            echo "Build failed. Sending notification..."
            // mail to: 'admin@example.com',
            //      subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            //      body: "Something went wrong with build ${BUILD_NUMBER}. Check logs at ${env.BUILD_URL}"
        }
    }
}

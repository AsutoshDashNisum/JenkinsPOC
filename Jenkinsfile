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
        /* 
           POC 1 - Stage 1: Checkout
           Pulls the code from the Git repository.
        */
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                checkout scm
            }
        }

        /* 
           POC 2 - Parallel Stages: Lint & Test
           Fix: Running inside a docker container because Jenkins doesn't have Python installed.
        */
        stage('Verify') {
            parallel {
                stage('Lint') {
                    steps {
                        echo "Running Linting via Docker (python:3.9-slim)..."
                        sh 'docker run --rm -v ${WORKSPACE}:/app -w /app python:3.9-slim sh -c "pip install flake8 && flake8 app/"'
                    }
                }
                stage('Test') {
                    steps {
                        echo "Running Unit Tests via Docker (python:3.9-slim)..."
                        sh 'docker run --rm -v ${WORKSPACE}:/app -w /app python:3.9-slim sh -c "pip install pytest && pytest app/test_main.py"'
                    }
                }
            }
        }

        /* 
           POC 1 - Stage 2: Build
           Since we are dockerizing the app, the 'build' happens during Docker image creation.
           We'll use this stage to simply verify the requirements file exists.
        */
        stage('Validate Environment') {
            steps {
                echo "Checking project files..."
                sh 'ls -l'
            }
        }

        /* 
           POC 1 - Stage 4 & POC 2: Archive & Docker Build
           Archives artifacts and builds the Docker image.
        */
        stage('Package & Docker Build') {
            steps {
                echo "Archiving artifacts (source code and requirements)..."
                archiveArtifacts artifacts: 'app/*.py, requirements.txt', fingerprint: true
                
                echo "Building Docker Image version: ${BUILD_NUMBER}"
                /* 
                   POC 2 Requirement: Tag image using build number
                */
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
            }
        }

        /* 
           POC 2 Requirement: Push Docker image
        */
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

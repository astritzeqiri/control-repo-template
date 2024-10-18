pipeline {
    agent any
    environment {
        LIBLAB_TOKEN = credentials('LIBLAB_TOKEN')
        LIBLAB_BITBUCKET_TOKEN = credentials('LIBLAB_BITBUCKET_TOKEN')
    }
    stages {
        stage('Checkout Repository') {
            steps {
                // Ensure the repository is checked out
                checkout scm
            }
        }
        stage('Install Node.js and npm') {
            steps {
                script {
                    // Download and install Node.js and npm
                    sh '''
                    curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                    sudo apt-get install -y nodejs
                    node -v
                    npm -v
                    '''
                }
            }
        }
        stage('Pre-check for Environment Variables') {
            steps {
                script {
                    if (!env.LIBLAB_TOKEN || !env.LIBLAB_BITBUCKET_TOKEN) {
                        error("Error: LIBLAB_TOKEN or LIBLAB_BITBUCKET_TOKEN is not defined")
                    }
                    echo "Environment variables are defined."
                }
            }
        }
        stage('Check for File Changes') {
            steps {
                script {
                    def changes = sh(script: 'git diff --quiet HEAD~1 -- liblab.config.json spec/ hooks/ customPlanModifiers/ || echo "CHANGES"', returnStdout: true).trim()
                    if (changes != "CHANGES") {
                        echo "No changes detected in relevant files. Skipping pipeline."
                        currentBuild.result = 'SUCCESS'
                        error("Stopping pipeline as no relevant files were changed")
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    if (!env.LIBLAB_TOKEN || !env.LIBLAB_BITBUCKET_TOKEN) {
                        error("Error: Variables LIBLAB_TOKEN and LIBLAB_BITBUCKET_TOKEN are required. Please ensure they are set.")
                    }
                    sh 'npm install -g liblab'
                }
            }
        }
        stage('Build SDKs and Create Pull Request') {
            steps {
                script {
                    sh 'liblab build --yes --pr -p bitbucket'
                }
            }
        }
    }
    post {
        always {
            // Clean up workspace after the pipeline is finished
            cleanWs()
        }
    }
}

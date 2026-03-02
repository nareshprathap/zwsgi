pipeline {
    agent any

    stages {

        stage('Generate Repo Global Build Number') {
            steps {
                script {
                    // 1️⃣ Identify repo name dynamically (from Jenkins job)
                    def repoName = "${env.JOB_NAME.split('/')[0]}" // e.g., "myapp"

                    // 2️⃣ Counter file for this repo
                    def counterFile = "${JENKINS_HOME}/global-build-number-${repoName}.txt"

                    // 3️⃣ Lock to prevent concurrent increments for this repo
                    lock("global-build-number-${repoName}-lock") {

                        // 4️⃣ Initialize counter if it doesn't exist
                        if (!fileExists(counterFile)) {
                            writeFile file: counterFile, text: "1"
                            echo "Counter file created for ${repoName} with initial value 1"
                        }

                        // 5️⃣ Read current number
                        def buildNumber = readFile(counterFile).trim().toInteger()

                        // 6️⃣ Increment and save for next build
                        writeFile file: counterFile, text: (buildNumber + 1).toString()

                        // 7️⃣ Set environment variables for build
                        env.GLOBAL_BUILD_NUMBER = buildNumber.toString()
                        env.RELEASE_VERSION = "1.0.${env.GLOBAL_BUILD_NUMBER}"
                        env.IMAGE_TAG = "${env.BRANCH_NAME.replaceAll('/', '-')}-${env.GLOBAL_BUILD_NUMBER}"

                        // 8️⃣ Update Jenkins display name
                        currentBuild.displayName = "#${env.GLOBAL_BUILD_NUMBER}"
                        echo "Repo: ${repoName}, Global Build Number: ${env.GLOBAL_BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Would run build here"'
            }
        }

        stage('Deploy') {
            steps {
                sh "echo 'Would deploy using RELEASE_VERSION=${env.RELEASE_VERSION}'"
            }
        }
    }

    post {
        success {
            echo "Build ${env.GLOBAL_BUILD_NUMBER} for ${env.BRANCH_NAME} completed successfully."
        }
        failure {
            echo "Build ${env.GLOBAL_BUILD_NUMBER} for ${env.BRANCH_NAME} failed!"
        }
        always {
            cleanWs()
        }
    }
}

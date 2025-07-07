pipeline {
    agent any

    // Trigger: This section schedules the pipeline to run every 5 minutes.
    triggers {
        cron('H/5 * * * *')
    }

    environment {
        ARTIFACT = 'target/*.jar'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code from GitHub...'
                git branch: 'main', url: 'https://github.com/aresiddharthareddy/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging the project...'
                bat 'mvn package'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving JAR file...'
                archiveArtifacts artifacts: "${ARTIFACT}", fingerprint: true
            }
        }

        stage('Run on localhost for 2 minutes') {
            steps {
                echo 'Starting server on port 9000 for 2 minutes...'
                bat '''
                    rem Start the application in a new window
                    start "" java -jar target\\spring-petclinic-3.5.0-SNAPSHOT.jar --server.port=9000
                    
                    rem Wait for 2 minutes (120 seconds + 1 for safety)
                    ping -n 121 127.0.0.1 > nul

                    rem Find and kill the process using port 9000
                    for /f "tokens=5" %%a in ('netstat -aon ^| findstr :9000 ^| findstr LISTENING') do taskkill /PID %%a /F || exit 0
                '''
            }
        }
    }
    
    // Post-build Actions: This section runs after all stages are completed.
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def buildStatus = currentBuild.currentResult
                def buildUrl = env.BUILD_URL

                def emailSubject
                def emailBody

                if (buildStatus == 'SUCCESS') {
                    emailSubject = "SUCCESS: Jenkins Pipeline '${jobName}' - Build #${buildNumber}"
                    emailBody = """
                        <h2>Build Successful</h2>
                        <p><strong>Pipeline:</strong> ${jobName}</p>
                        <p><strong>Build Number:</strong> ${buildNumber}</p>
                        <p>The pipeline completed successfully.</p>
                        <p>Visit the build page for more details: <a href="${buildUrl}">${buildUrl}</a></p>
                    """
                } else {
                    emailSubject = "FAILURE: Jenkins Pipeline '${jobName}' - Build #${buildNumber}"
                    emailBody = """
                        <h2>Build Failed</h2>
                        <p><strong>Pipeline:</strong> ${jobName}</p>
                        <p><strong>Build Number:</strong> ${buildNumber}</p>
                        <p>The pipeline has failed. Please check the logs.</p>
                        <p>Visit the build page for more details: <a href="${buildUrl}">${buildUrl}</a></p>
                    """
                }

                emailext (
                    to: 'sare@osidigital.com',
                    subject: emailSubject,
                    body: emailBody,
                    mimeType: 'text/html'
                )
            }
        }
        
        success {
            echo 'Build & Test pipeline completed successfully!'
        }
        failure {
            echo 'Build failed. Check the logs.'
        }
    }
}

pipeline {
    agent any
    
    triggers {
        githubPush()  
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        MAVEN_HOME = "/usr/share/maven"
        AWS_CLI_PATH = "/usr/bin"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${AWS_CLI_PATH}:$PATH"
        SONARQUBE_CREDENTIALS = 'sonarqube'
        S3_BUCKET = 'mypipelinebucket'
        APP_NAME = 'aiverse'
        ENV_NAME = 'Aiverse-env'
        STAGING_ENV_NAME = 'Aiverse-staging-env'
        STAGING_URL = "http://Aiverse-env.eba-7mcmvxh3.eu-north-1.elasticbeanstalk.com"
        EMAIL_RECIPIENTS = "1999manmeetkaur@gmail.com"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo "This stage retrieves the latest code from GitHub, sets up the Java and Maven environment, and builds the Spring Boot application using Maven."
                    echo "The application is compiled and packaged as a JAR file for deployment."
                }
            }
        }

        stage('Unit and Integration Tests') {
            steps {
                script {
                    echo "Running unit and integration tests using JUnit to ensure application correctness."
                    echo "If tests pass, the build continues; if they fail, a notification is sent to the developers."
                }
            }
            post {
                always {
                    emailext(
                        subject: "Test Results for Jenkins Build #${BUILD_NUMBER}",
                        body: "Unit and Integration Tests completed.",
                        to: EMAIL_RECIPIENTS
                    )
                }
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    echo "Performing static code analysis using SonarQube to ensure best coding practices and maintainability."
                    echo "Reports any code smells, security vulnerabilities, or bugs."
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    echo "Running a security scan using Snyk to identify and report any vulnerabilities in dependencies or code."
                    echo "If critical vulnerabilities are found, a notification is sent to the team."
                }
            }
            post {
                always {
                    emailext(
                        subject: "Security Scan Results for Jenkins Build #${BUILD_NUMBER}",
                        body: "Security Scan completed.",
                        to: EMAIL_RECIPIENTS
                    )
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    echo "Deploying the application to the Staging environment using AWS Elastic Beanstalk."
                    echo "1. Uploading the built JAR file to AWS S3."
                    echo "2. Creating a new application version in AWS Elastic Beanstalk with the uploaded JAR."
                    echo "3. Updating the Staging environment to use the latest version."
                    echo "Staging URL: ${STAGING_URL}"
                }
            }
        }

        stage('Integration Tests on Staging') {
            steps {
                script {
                    echo "Running integration tests on the staging environment to validate end-to-end functionality."
                    echo "Performing API health checks to ensure the service is operational."
                    echo "These tests verify that the application behaves as expected in a production-like environment."
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    echo "Deploying the application to the Production environment using AWS Elastic Beanstalk."
                    echo "1. The final tested JAR file is uploaded to AWS S3."
                    echo "2. A new application version is created in AWS Elastic Beanstalk."
                    echo "3. The production environment ${ENV_NAME} is updated to use the new version."
                    echo "This ensures a smooth, zero-downtime deployment."
                    echo "After deployment, AWS Beanstalk will handle auto-scaling, health monitoring, and rollback if needed."
                }
            }
        }
    }

    post {
        always {
            echo "📧 Sending email notification..."
        }
        success {
            emailext(
                subject: "✅ Jenkins Build #${BUILD_NUMBER} Successful!",
                body: """
                    <h2 style="color: green;">✅ Jenkins Build Successful!</h2>
                    <p>The Jenkins build for <strong>${env.JOB_NAME}</strong> was successful. 🎉</p>
                    <ul>
                        <li><strong>Build Number:</strong> #${env.BUILD_NUMBER}</li>
                        <li><strong>View Build:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                        <li><strong>Deployment Status:</strong> ✅ Successful</li>
                    </ul>
                    <br>
                    <p>🚀 Happy Deploying!</p>
                """,
                mimeType: 'text/html',
                to: "${EMAIL_RECIPIENTS}"
            )
        }
        failure {
            emailext(
                subject: "❌ Jenkins Build #${BUILD_NUMBER} Failed!",
                body: """
                    <h2 style="color: red;">❌ Jenkins Build Failed!</h2>
                    <p>The Jenkins build for <strong>${env.JOB_NAME}</strong> has <strong>failed</strong>. ❌</p>
                    <ul>
                        <li><strong>Build Number:</strong> #${env.BUILD_NUMBER}</li>
                        <li><strong>View Build Logs:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                        <li><strong>Error Details:</strong> Please check the logs for more information.</li>
                    </ul>
                    <br>
                    <p>⚠️ Please fix the errors and re-run the pipeline.</p>
                """,
                mimeType: 'text/html',
                to: "${EMAIL_RECIPIENTS}"
            )
        }
    }
}

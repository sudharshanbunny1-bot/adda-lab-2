pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        DEPLOY_DIR = 'C:\\SpringBootApp'
        TASK_NAME = 'SpringBootApp'
        APP_PORT = '8081'
    }

    stages {

        stage('Stop Application') {
            steps {
                bat '''
                for /f "tokens=5" %%a in ('netstat -ano ^| findstr :8081 ^| findstr LISTENING') do (
                    echo Stopping process %%a on port 8081...
                    taskkill /F /PID %%a
                )
                exit /b 0
                '''
            }
        }

        stage('Build') {
            steps {
                bat 'mvnw.cmd clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                if not exist "%DEPLOY_DIR%" mkdir "%DEPLOY_DIR%"
                copy /Y "target\\demo-0.0.1-SNAPSHOT.jar" "%DEPLOY_DIR%\\app.jar"
                if errorlevel 1 exit /b 1
                echo New JAR deployed successfully.
                '''
            }
        }

        stage('Start Application') {
            steps {
                bat '''
                schtasks /run /tn "%TASK_NAME%"
                echo Waiting for Spring Boot to start...
                ping -n 16 127.0.0.1 > nul
                netstat -ano | findstr :8081 | findstr LISTENING
                if errorlevel 1 (
                    echo ERROR: Application failed to start!
                    type %DEPLOY_DIR%\\app.log
                    exit /b 1
                )
                echo Spring Boot started successfully!
                '''
            }
        }
    }

    post {
        success {
            echo 'BUILD AND DEPLOYMENT SUCCESSFUL!'
        }
        failure {
            echo 'PIPELINE FAILED! Check Jenkins console output.'
        }
    }
}

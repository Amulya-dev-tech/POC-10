pipeline {
    agent any

    environment {
        APP_NAME     = "myapp"
        DOCKER_IMAGE = "myapp-image"
    }

    tools {
        // Must match the name configured in Manage Jenkins → Global Tool Configuration → Maven
        maven 'maven-3.9.11'
        // If your build needs a specific JDK tool, add:
        // jdk 'jdk-17'  // Ensure tool exists, or remove this line
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC_3.git'
            }
        }

        stage('Maven Build') {
            steps {
                // With the tools block, Jenkins prepends MAVEN_HOME/bin to PATH, so 'mvn' is available
                sh 'mvn -V -B clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 8087:8080 ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success { echo "Pipeline executed successfully!" }
        failure { echo "Pipeline failed!" }
    }
}

pipeline {
    agent any

    environment {
        APP_NAME     = 'myapp'
        DOCKER_IMAGE = 'myapp-image'
        // If you need to push to a registry, add registry envs here, e.g.:
        // DOCKER_REGISTRY = 'registry.example.com'
        // DOCKER_CREDENTIALS_ID = 'docker-creds'
    }

    tools {
        // Must match names configured in:
        // Manage Jenkins → Global Tool Configuration
        maven 'maven-3.8.4'
        // Uncomment only if you have a configured JDK tool
        // jdk 'jdk-17'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC-10.git'
            }
        }

        stage('Maven Build') {
            steps {
                // MAVEN_HOME/bin is added to PATH via the tools directive, so 'mvn' is available
                sh 'mvn -V -B clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // 'SonarQube' must match the name under Manage Jenkins → Configure System → SonarQube servers
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B sonar:sonar'
                }
            }
        }

        // Optional: uncomment if you want the pipeline to wait for SonarQube Quality Gate
        // stage('Quality Gate') {
        //     steps {
        //         timeout(time: 10, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        stage('Docker Build') {
            steps {
                // Ensure the agent has Docker installed and daemon is running
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Example: collect artifacts or clean up
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
            // sh 'docker logs ${APP_NAME} || true'
        }
    }
}

pipeline {
    agent any

    environment {
        APP_NAME        = "myapp"
        DOCKER_IMAGE    = "myapp-image"
        // SonarQube project metadata (adjust to your org/project)
        SONAR_PROJECT_KEY = "myorg_myapp"
        SONAR_PROJECT_NAME = "MyApp"
        SONAR_PROJECT_VERSION = "1.0.${BUILD_NUMBER}"
        // Name must match Jenkins → Configure System → SonarQube servers (e.g., "SonarQube")
        SONARQUBE_SERVER = "SonarQube"
    }

    tools {
        maven 'maven-3.9.11'
        // If you have configured a JDK tool, uncomment:
        // jdk 'jdk-17'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC_3.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn -V -B clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Bind SonarQube server environment configured in Jenkins
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    // Use a SonarQube token stored as Jenkins Secret Text credential
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        // For Maven projects, run the sonar goal.
                        // Ensure your pom.xml has <sonar-maven-plugin> or use the default via org.sonarsource.scanner.maven:sonar-maven-plugin
                        sh """
                            mvn -B sonar:sonar \
                              -Dsonar.projectKey='${SONAR_PROJECT_KEY}' \
                              -Dsonar.projectName='${SONAR_PROJECT_NAME}' \
                              -Dsonar.projectVersion='${SONAR_PROJECT_VERSION}' \
                              -Dsonar.host.url="$SONAR_HOST_URL" \
                              -Dsonar.login="$SONAR_TOKEN"
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            // This requires the "Wait for Quality Gate" step from the SonarQube plugin
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Fails the pipeline if Quality Gate status is failed
                    waitForQualityGate()
                }
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
        always {
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
    }
}

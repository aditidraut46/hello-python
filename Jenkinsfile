pipeline {
    agent any

    environment {
        // Set Maven tool if needed
        MAVEN_HOME = tool 'Maven 3'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                  echo "Building project..."
                  mvn clean install
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      echo "Running SonarQube scan..."
                      sonar-scanner \
                        -Dsonar.projectKey=hello-python \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://34.58.236.251:9000 \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  echo "Deploying application..."
                  # put your deployment steps here
                '''
            }
        }
    }
}


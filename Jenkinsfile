pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube'
        SCANNER = 'SonarScanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/aditidraut46/hello-python.git'
            }
        }

        stage('Install & Run Tests') {
            steps {
                sh '''
                  echo "📦 Installing dependencies..."
                  python3 -m pip install --upgrade pip
                  pip3 install --user -r requirements.txt
                  
                  echo "🧪 Running tests..."
                  python3 -m pytest -q
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    withEnv(["PATH+SONAR=${tool SCANNER}/bin"]) {
                        sh '''
                          echo "🔍 Running SonarQube scan..."
                          sonar-scanner \
                            -Dsonar.projectKey=hello-python \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.python.version=3.10
                        '''
                    }
                }
            }
        }

        stage('Deploy to App VM') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        echo "🚀 Deploying with systemd..."

                        ssh -o StrictHostKeyChecking=no aditidraut46@34.16.36.23 "mkdir -p /home/aditidraut46/app"
                        scp -o StrictHostKeyChecking=no -r * aditidraut46@34.16.36.23:/home/aditidraut46/app/

                        ssh -o StrictHostKeyChecking=no aditidraut46@34.16.36.23 "
                          sudo systemctl daemon-reload &&
                          sudo systemctl restart flaskapp &&
                          sudo systemctl enable flaskapp
                        "

                        echo "✅ Deployment complete. Check: curl http://34.16.36.23:8080"
                    '''
                }
            }
        }

        stage('Quality Gate (Async)') {
            steps {
                script {
                    // Run Quality Gate check asynchronously
                    def qg = waitForQualityGate(timeout: 5, abortPipeline: false)
                    echo "🔔 SonarQube Quality Gate status: ${qg.status}"
                    
                    if (qg.status != 'OK') {
                        echo "⚠️ Quality Gate failed, but deployment already ran. Investigate manually."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline Succeeded"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}


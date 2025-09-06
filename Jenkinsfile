pipeline {
    agent any

    environment {
        PATH = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/var/lib/jenkins/.local/bin"
        PYTHONPATH = "."
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    python3 -m pip install --upgrade pip
                    pip3 install -r requirements.txt
                    pytest -q
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') { // Must match your Jenkins SonarQube server config
                    withEnv(["PATH+SONAR=${tool 'SonarScanner'}/bin"]) { // Use Jenkins SonarScanner tool
                        sh '''
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

        stage('Check Quality Gate (Non-blocking)') {
            steps {
                script {
                    try {
                        timeout(time: 15, unit: 'MINUTES') {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                echo "⚠️ Quality Gate status: ${qg.status} — Deployment will continue."
                            } else {
                                echo "✅ Quality Gate passed"
                            }
                        }
                    } catch (err) {
                        echo "⚠️ Could not get Quality Gate status in time: ${err} — Deployment will continue."
                    }
                }
            }
        }

        stage('Deploy to App VM') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r * aditidraut46@34.30.82.60:/home/aditidraut46/app/
                        ssh -o StrictHostKeyChecking=no aditidraut46@34.30.82.60 '
                            pkill -f "python3 /home/aditidraut46/app/app.py" || true
                            nohup python3 /home/aditidraut46/app/app.py > /home/aditidraut46/app/app.log 2>&1 &
                        '
                    '''
                }
            }
        }
    }

    post {
        success { echo "Pipeline Succeeded" }
        failure { echo "Pipeline Failed" }
    }
}


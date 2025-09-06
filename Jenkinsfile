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

        stage('SonarQube Analysis (Non-blocking)') {
            steps {
                withSonarQubeEnv('sonarqube') { 
                    withEnv(["PATH+SONAR=${tool 'SonarScanner'}/bin"]) { 
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

        stage('Test SSH Connection') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh 'ssh -o StrictHostKeyChecking=no aditidraut46@35.202.26.230 "echo Connected!"'
                }
            }
        }

        stage('Deploy to App VM') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r * aditidraut46@35.202.26.230:/home/aditidraut46/app/
                        ssh -o StrictHostKeyChecking=no aditidraut46@35.202.26.230 '
                            pkill -f "python3 /home/aditidraut46/app/app.py" || true
                            nohup python3 /home/aditidraut46/app/app.py > /home/aditidraut46/app/app.log 2>&1 &
                        '
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ Pipeline Succeeded" }
        failure { echo "❌ Pipeline Failed, check logs!" }
    }
}
 

pipeline {
  agent any

  stages {
    stage('Checkout') { 
      steps { checkout scm } 
    }

    stage('Run Tests') {
      steps {
        sh '''
          python3 -m pip install --upgrade pip
          pip3 install -r requirements.txt
          export PATH=$PATH:/var/lib/jenkins/.local/bin
          export PYTHONPATH=.
          pytest -q
        '''
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          withEnv(["PATH+SONAR=${tool 'SonarScanner'}/bin"]) {
            sh '''
              sonar-scanner \
                -Dsonar.projectKey=hello-python \
                -Dsonar.sources=. \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_AUTH_TOKEN
            '''
          }
        }
      }
    }

    stage('Wait Quality Gate') {
      steps {
        timeout(time:11, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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
    success { echo " Pipeline Succeeded" }
    failure { echo " Pipeline Failed" }
  }
}


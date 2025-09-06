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
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
          withSonarQubeEnv('sonarqube') {
            sh '''
              sonar-scanner \
                -Dsonar.projectKey=hello-python \
                -Dsonar.sources=. \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_TOKEN \
                -Dsonar.python.version=3.10
            '''
          }
        }
      }
    }

    stage('Wait Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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
    failure { echo "❌ Pipeline Failed" }
  }
}


pipeline { 
  agent any

  environment {
    DOCKER_IMAGE = 'sigr-restaurante'
    DEPLOY_SERVER = credentials('deploy-server-ssh')
  }
  stages { stage('Checkout') {
      steps { checkout scm
      sh 'echo "Rama: ${GIT_BRANCH}"'
    }
  }

  stage('Install & Lint') { parallel {
      stage('Backend') { steps {
        dir('backend') { sh 'npm ci'
          sh 'npm run lint'
        }
      }	
    }
    stage('Frontend') { steps {
        dir('frontend') { sh 'npm ci'
            sh 'npm run lint'
        }
      }
    }
  }
}

stage('Test') { steps {
    dir('backend') {
      sh 'npm test -- --coverage'
    }
  }
  post {
    always {
      junit 'backend/coverage/junit.xml' publishHTML target: [
      reportDir: 'backend/coverage/lcov-report', reportFiles: 'index.html',
      reportName: 'Coverage Report'
      ]
    }
  }
}

stage('Build') { steps {
    sh 'docker compose build --no-cache'
  }
}

  stage('Deploy') {
      when { branch 'main' } steps {
        sshagent(['deploy-server-ssh']) { sh '''
        ssh -o StrictHostKeyChecking=no deploy@$DEPLOY_SERVER \ "cd /opt/sigr && git pull && docker compose up -d"
        '''
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline exitoso. Linea base estable.'
    }
    failure {
      echo 'Pipeline fallido. Revisar logs.'
    }
  }
}


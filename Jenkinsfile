pipeline {
  agent any

  environment {
    GIT_REPO = 'https://github.com/tej-wolf/simple_app.git'
  }

  options {
    skipDefaultCheckout(true)
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.GIT_REPO]]
        ])
      }
    }

    stage('Build') {
      steps {
        script {
          echo "Building project..."
        }
      }
    }

    stage('Unit Tests') {
      steps {
        script {
          echo "Running tests..."
        }
      }
    }

    stage('Docker Build (optional)') {
      when {
        expression { return fileExists('Dockerfile') }
      }
      steps {
        script {
          echo "Dockerfile found — building image"
          sh "docker build -t simple_app:${env.BUILD_NUMBER} ."
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          echo "Deployment placeholder — no actions yet"
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline completed SUCCESS"
    }
    failure {
      echo "Pipeline completed FAILURE"
    }
    always {
      cleanWs()
    }
  }
}

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
  
stage('Trivy Scan') {
  when { expression { return fileExists('Dockerfile') } }
  steps {
    script {
      def img = "simple_app:${env.BUILD_NUMBER}"
      // build image earlier in pipeline or ensure it exists
      sh """
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/workdir \
          aquasec/trivy:latest image --format json --output /workdir/trivy-report.json \
          --severity HIGH,CRITICAL --exit-code 1 ${img}
      """
      archiveArtifacts artifacts: 'trivy-report.json', onlyIfSuccessful: true
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

pipeline {
  agent any

  environment {
    // Edit these if you use them. Leave or remove if not needed.
    GIT_REPO = 'https://github.com/tej-wolf/simple_app.git'
    // DOCKER_REGISTRY = 'your.registry.com'
    // DOCKER_CREDENTIALS = 'docker-creds-id'
  }

  options {
    skipDefaultCheckout(true)
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        // Simple git checkout; adjust branch if required (main/master)
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.GIT_REPO]]
        ])
      }
    }

    stage('Build') {
      steps {
        echo "Building project..."
        // Change to the real build command for your project.
        // If not a Maven project, replace with npm, python, go, etc.
        sh '''
          if [ -f pom.xml ]; then
            mvn -B -DskipTests clean package
          else
            echo "No pom.xml found — skipping Maven build"
          fi
        '''
      }
    }

    stage('Unit Tests') {
      steps {
        echo "Running unit tests..."
        sh '''
          if [ -f pom.xml ]; then
            mvn test || true
          else
            echo "No tests to run (no pom.xml)"
          fi
        '''
      }
    }

    stage('Docker Build (optional)') {
      when {
        expression { return fileExists('Dockerfile') }
      }
      steps {
        script {
          echo "Dockerfile detected — building image (disabled push by default)"
          def img = "simple_app:${env.BUILD_NUMBER}"
          sh "docker build -t ${img} ."
          // To push, uncomment and configure credentials
          // withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, passwordVariable: 'PWD', usernameVariable: 'USER')]) {
          //   sh "docker login -u $USER -p $PWD ${env.DOCKER_REGISTRY}"
          //   sh "docker tag ${img} ${env.DOCKER_REGISTRY}/${img}"
          //   sh "docker push ${env.DOCKER_REGISTRY}/${img}"
          // }
        }
      }
    }

    stage('Deploy (placeholder)') {
      steps {
        echo "Deploy step — add your deploy commands here"
      }
    }
  }

  post {
    success {
      echo "Pipeline finished SUCCESS"
    }
    failure {
      echo "Pipeline finished FAILURE"
    }
    always {
      cleanWs()
    }
  }
}

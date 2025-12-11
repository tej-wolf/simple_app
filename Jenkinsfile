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
          branches: [[name: '*/main']], // change to '*/master' if needed
          userRemoteConfigs: [[url: env.GIT_REPO]]
        ])
      }
    }

    stage('Build') {
      steps {
        echo "Building project..."
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
      when { expression { return fileExists('Dockerfile') } }
      steps {
        script {
          env.IMAGE_NAME = "simple_app:${env.BUILD_NUMBER}"
          sh "docker build -t ${env.IMAGE_NAME} ."
        }
      }
    }

    // ===== Trivy scan + convert to SARIF + publish Warnings NG =====
    stage('Trivy Scan & Warnings') {
      when { expression { return fileExists('Dockerfile') } }
      steps {
        script {
          // image name built earlier
          def img = env.IMAGE_NAME ?: "simple_app:${env.BUILD_NUMBER}"

          // Run trivy image scan -> JSON (do not fail the pipeline here; we want to publish warnings)
          sh """
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/workdir \\
              aquasec/trivy:latest image --format json --output /workdir/trivy-report.json \\
              --severity HIGH,CRITICAL --exit-code 0 ${img} || true
          """

          // Convert trivy JSON -> SARIF (Warnings NG supports SARIF format)
          sh """
            docker run --rm -v \$(pwd):/workdir aquasec/trivy:latest convert \\
              --format sarif --output /workdir/trivy-report.sarif /workdir/trivy-report.json || true
          """

          // Archive reports so they are downloadable from the build
          archiveArtifacts artifacts: 'trivy-report.json, trivy-report.sarif', onlyIfSuccessful: false
        }
      }

      post {
        always {
          // Publish the SARIF into Warnings NG (requires Warnings Next Generation plugin)
          // Make sure Warnings NG plugin is installed in Jenkins.
          recordIssues tools: [sarif(pattern: 'trivy-report.sarif')]
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
    success { echo "Pipeline finished SUCCESS" }
    failure { echo "Pipeline finished FAILURE" }
    always { cleanWs() }
  }
}

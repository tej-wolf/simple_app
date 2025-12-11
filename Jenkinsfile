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
          branches: [[name: '*/main']], // change to master if you use master
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
      when {
        expression { return fileExists('Dockerfile') }
      }
      steps {
        script {
          def img = "simple_app:${env.BUILD_NUMBER}"
          sh "docker build -t ${img} ."
        }
      }
    }

    // ===== Trivy scan stage - properly inside 'stages' =====
    stage('Trivy Scan') {
      when {
        expression { return fileExists('Dockerfile') } // scan only if Dockerfile exists / image built
      }
      steps {
        script {
          def img = "simple_app:${env.BUILD_NUMBER}"
          // Run Trivy official Docker image to scan the local image and output JSON report to workspace
          sh """
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/workdir \\
              aquasec/trivy:latest image --format json --output /workdir/trivy-report.json \\
              --severity HIGH,CRITICAL --exit-code 1 ${img} || true
          """
          // Print a compact summary (if jq available), otherwise print count fallback
          sh '''
            if command -v jq >/dev/null 2>&1; then
              echo "Trivy summary:"
              jq -r '.Results[] | {Target:.Target, Vulnerabilities:(.Vulnerabilities|length)}' trivy-report.json || true
            else
              echo "jq not available — printing raw report head"
              head -n 200 trivy-report.json || true
            fi
          '''
          // Archive the JSON report for later inspection
          archiveArtifacts artifacts: 'trivy-report.json', onlyIfSuccessful: true
        }
      }
    }

    stage('Deploy (placeholder)') {
      steps {
        echo "Deploy step — add your deploy commands here"
      }
    }
  }
  
  stage('Trivy Scan & Warnings') {
    when {
      expression { return fileExists('Dockerfile') }
  }
    steps {
      script {
        def img = "simple_app:${env.BUILD_NUMBER}"

      // Run Trivy scan and produce JSON output
      sh """
        docker run --rm \
         -v /var/run/docker.sock:/var/run/docker.sock \
         -v \$(pwd):/workdir \
         aquasec/trivy:latest image \
         --format json \
         --output /workdir/trivy-report.json \
         --severity HIGH,CRITICAL \
         --exit-code 0 \
         ${img}
      """

      // Convert JSON → SARIF (Warnings NG supports SARIF)
      sh """
        docker run --rm \
          -v \$(pwd):/workdir \
          aquasec/trivy:latest convert \
          --format sarif \
          --output /workdir/trivy-report.sarif \
          /workdir/trivy-report.json
      """

      // Archive scan reports
      archiveArtifacts artifacts: 'trivy-report.json', onlyIfSuccessful: true
      archiveArtifacts artifacts: 'trivy-report.sarif', onlyIfSuccessful: true
    }
  }
  post {
    always {
      // Publish Trivy warnings in Jenkins UI (requires Warnings NG plugin)
      recordIssues tools: [sarif(pattern: 'trivy-report.sarif')]
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

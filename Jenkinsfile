pipeline {
  agent any

  environment {
    GIT_REPO = 'https://github.com/tej-wolf/simple_app.git'
  }

  options { skipDefaultCheckout(true); timestamps() }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: env.GIT_REPO]]])
      }
    }

    stage('Build') {
      steps {
        script {
          if (fileExists('pom.xml')) {
            if (isUnix()) {
              sh "mvn -B -DskipTests clean package"
            } else {
              bat "mvn -B -DskipTests clean package"
            }
          } else {
            echo "No pom.xml - skipping build"
          }
        }
      }
    }

    stage('Docker Build') {
      when { expression { return fileExists('Dockerfile') } }
      steps {
        script {
          env.IMAGE = "simple_app:${env.BUILD_NUMBER}"
          if (isUnix()) {
            sh "docker build -t ${env.IMAGE} ."
          } else {
            // Windows: docker must be installed and in PATH (Docker Desktop)
            bat "docker build -t ${env.IMAGE} ."
          }
        }
      }
    }

    stage('Trivy Scan & Publish') {
      when { expression { return fileExists('Dockerfile') } }
      steps {
        script {
          if (isUnix()) {
            sh """
              docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/workdir \\
                aquasec/trivy:latest image --format json --output /workdir/trivy-report.json \\
                --severity HIGH,CRITICAL --exit-code 0 ${env.IMAGE} || true

              docker run --rm -v \$(pwd):/workdir aquasec/trivy:latest convert \\
                --format sarif --output /workdir/trivy-report.sarif /workdir/trivy-report.json || true
            """
          } else {
            // Windows: mounts and docker socket handling differ; rely on docker CLI.
            bat """
              docker run --rm -v %cd%:/workdir aquasec/trivy:latest image --format json --output /workdir/trivy-report.json --severity HIGH,CRITICAL --exit-code 0 ${env.IMAGE} || exit 0
              docker run --rm -v %cd%:/workdir aquasec/trivy:latest convert --format sarif --output /workdir/trivy-report.sarif /workdir/trivy-report.json || exit 0
            """
          }
          archiveArtifacts artifacts: 'trivy-report.json, trivy-report.sarif', onlyIfSuccessful: false
        }
      }
      post {
        always { recordIssues tools: [sarif(pattern: 'trivy-report.sarif')] }
      }
    }

    stage('Deploy (placeholder)') {
      steps { echo "Deploy placeholder" }
    }
  }

  post { always { cleanWs() } }
}

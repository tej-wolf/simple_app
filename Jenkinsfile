pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                echo "No tests yet!"
            }
        }

        stage('Run App') {
            steps {
                echo "Starting Flask app"
                sh 'python app.py &'
            }
        }
    }
}

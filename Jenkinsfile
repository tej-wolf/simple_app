pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tej-wolf/simple_app.git'
            }
        }

        stage('Install') {
            steps {
                sh '''
                    python3 -m venv venv          # create virtual environment
                    . venv/bin/activate           # activate it
                    pip install --upgrade pip     # upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }

    }
}

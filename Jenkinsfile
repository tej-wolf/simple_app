pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tej-wolf/simple_app.git'
            }
        }

stage('Install Dependencies') {
  steps {
    sh '''
      python -m venv .venv
      . .venv/bin/activate
      python -m pip install --upgrade pip
      pip install -r requirements.txt
    '''
  }
}

stage('Run Tests') {
  steps {
    sh '''
      . .venv/bin/activate
      # run your tests here, example:
      # pytest -q
      echo "No tests yet"
    '''
  }
}

stage('Run App') {
  steps {
    sh '''
      . .venv/bin/activate
      # run app in background for demo (not for production)
      python app.py &
    '''
  }
}

       

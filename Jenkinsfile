pipeline {
    agent any

    triggers {
            cron('H/5 * * * *')
            githubPush()
        }
        
    environment {
        HOST       = '0.0.0.0'
        PORT       = '5000'
        APP_MODULE = 'app:app'     // ⚠️ à adapter : fichier:objet Flask
        REPO_URL   = 'https://github.com/NABILAESSAIDY/Jenkins-CI-CD--.git'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: "${REPO_URL}"
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    python --version
                '''
            }
        }

        stage('Run Tests') {
            when {
                not {
                    changeset "README.md"
                }
            }
            parallel {
                stage('Test app 1') {
                    steps {
                        sh '''
                            . venv/bin/activate
                            COVERAGE_FILE=.coverage_1 pytest test_app.py -v --cov=. --cov-report=term
                        '''
                    }
                }
                stage('Test app 2') {
                    steps {
                        sh '''
                            . venv/bin/activate
                            COVERAGE_FILE=.coverage_2 pytest test_app_2.py -v --cov=. --cov-report=term
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    . venv/bin/activate
                    pkill -f gunicorn || true
                    JENKINS_NODE_COOKIE=dontKillMe BUILD_ID=dontKillMe \
                    nohup gunicorn --bind ${HOST}:${PORT} ${APP_MODULE} > gunicorn.log 2>&1 &
                    sleep 3
                    cat gunicorn.log
                '''
            }
        }
    }
}
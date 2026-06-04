pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/andresfalla/todo-list-aws.git'

                sh '''
                    echo "Descargando configuración de PRODUCTION..."
                    curl -o samconfig.toml \
                    "https://raw.githubusercontent.com/andresfalla/todo-list-aws-config/production/samconfig.toml?nocache=$(date +%s)"

                    mkdir -p .aws-sam
                    cp samconfig.toml .aws-sam/samconfig.toml

                    echo "Contenido del samconfig.toml:"
                    cat samconfig.toml
                '''
            }
        }

        stage('Static Test') {
            steps {
                sh '''
                    mkdir -p reports
                    flake8 src --format=html --htmldir=reports/flake8
                    bandit -r src -f html -o reports/bandit.html
                '''
            }
            post {
                always {
                    publishHTML(target: [
                        reportDir: 'reports/flake8',
                        reportFiles: 'index.html',
                        reportName: 'Flake8 Report'
                    ])
                    publishHTML(target: [
                        reportDir: 'reports',
                        reportFiles: 'bandit.html',
                        reportName: 'Bandit Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    export AWS_DEFAULT_REGION=us-east-1
                    sam build
                    sam validate --region us-east-1
                    sam deploy --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL="https://8h7c9yanla.execute-api.us-east-1.amazonaws.com/Prod"
                    pytest test/integration --junitxml=result-rest.xml
                '''
            }
            post {
                always {
                    junit 'result-rest.xml'
                }
            }
        }

    }
}
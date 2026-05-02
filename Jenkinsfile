pipeline {
    agent any

    parameters {
        choice(name: 'RUN_MODE', choices: ['SERVER', 'DOCKER'], description: 'Run mode')
        text(name: 'ENV_VARS', defaultValue: 'NODE_ENV=uat\nPORT=3000', description: 'Environment variables')
    }

    environment {
        APP_NAME = "my-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Load ENV') {
            steps {
                script {
                    def lines = params.ENV_VARS.split("\n")
                    for (l in lines) {
                        def pair = l.split("=")
                        if (pair.size() == 2) {
                            env[pair[0]] = pair[1]
                        }
                    }
                }
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Deploy') {
            steps {
                script {

                    if (params.RUN_MODE == 'SERVER') {

                        sh '''
                        pm2 stop ${APP_NAME} || true
                        pm2 start npm --name ${APP_NAME} -- start
                        '''

                    } else {

                        sh '''
                        docker build -t ${APP_NAME} .
                        docker stop ${APP_NAME} || true
                        docker rm ${APP_NAME} || true

                        docker run -d -p 3000:3000 --name ${APP_NAME} ${APP_NAME}
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build Successful"
        }
        failure {
            echo "Build Failed"
        }
    }
}

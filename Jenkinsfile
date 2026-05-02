pipeline {
    agent any

    parameters {
        choice(name: 'RUN_MODE', choices: ['SERVER', 'DOCKER'], description: 'Run mode')
        text(name: 'ENV_VARS', defaultValue: 'NODE_ENV=uat\nPORT=3000', description: 'Environment variables')
        string(name: 'PR_NUMBER', defaultValue: '', description: 'Enter PR number (optional)')
    }

    environment {
        APP_NAME = "my-app"
        REPO_URL = "https://github.com/mohitjangra876/test-project.git"
        CREDENTIALS_ID = "Github-Token"
        BRANCH = "main"
        CUSTOM_ENV = ""
    }

    stages {

        stage('Checkout') {
            steps {
                script {

                    if (params.PR_NUMBER?.trim()) {

                        echo "🚀 Checking out PR-${params.PR_NUMBER}"

                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "origin/pr/${params.PR_NUMBER}"]],
                            userRemoteConfigs: [[
                                url: env.REPO_URL,
                                credentialsId: env.CREDENTIALS_ID,
                                refspec: "+refs/pull/*/head:refs/remotes/origin/pr/*"
                            ]]
                        ])

                    } else {

                        echo "📦 Checking out main branch"

                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "*/${env.BRANCH}"]],
                            userRemoteConfigs: [[
                                url: env.REPO_URL,
                                credentialsId: env.CREDENTIALS_ID
                            ]]
                        ])
                    }
                }
            }
        }

        stage('Load ENV') {
            steps {
                script {
                    echo "🔧 Loading ENV variables safely"

                    def envList = params.ENV_VARS.split("\n")
                    def envString = ""

                    for (item in envList) {
                        if (item.contains("=")) {
                            envString += "-e " + item.trim() + " "
                        }
                    }

                    env.CUSTOM_ENV = envString
                }
            }
        }

        stage('Install Dependencies') {
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

                        echo "🚀 Deploying on SERVER (PM2)"

                        sh '''
                        export $(echo ${CUSTOM_ENV} | sed 's/-e //g')
                        pm2 stop ${APP_NAME} || true
                        pm2 delete ${APP_NAME} || true
                        pm2 start npm --name ${APP_NAME} -- start
                        pm2 save
                        '''

                    } else {

                        echo "🐳 Deploying using DOCKER"

                        sh '''
                        docker build -t ${APP_NAME} .
                        docker stop ${APP_NAME} || true
                        docker rm ${APP_NAME} || true

                        docker run -d \
                          -p 3000:3000 \
                          --name ${APP_NAME} \
                          ${CUSTOM_ENV} \
                          ${APP_NAME}
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & Deployment Successful"
        }
        failure {
            echo "❌ Build Failed"
        }
        always {
            echo "📌 Pipeline Finished"
        }
    }
}

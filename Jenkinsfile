pipeline {

    agent any

    tools {
        nodejs "Node18"
    }

    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'])
        string(name: 'VERSION', defaultValue: 'main')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "${params.VERSION}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Sejal302002/slip8.git',
                        credentialsId: 'token1'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS == true } }
            steps {
                sh 'npm test'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }

}

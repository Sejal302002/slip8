pipeline {

    // Use Node.js Docker Image so `npm` is always available
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root'
        }
    }

    // Pipeline parameters
    parameters {

        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run unit tests before deployment'
        )

        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'staging', 'prod'],
            description: 'Deployment environment'
        )

        string(
            name: 'VERSION',
            defaultValue: 'main',
            description: 'Git branch or tag to checkout'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                echo '=========================================='
                echo 'Stage: Checkout'
                echo "Version (Branch/Tag): ${params.VERSION}"
                echo '=========================================='

                script {

                    if (params.VERSION.startsWith('v') || params.VERSION.matches(/^\\d+\\.\\d+\\.\\d+$/)) {
                        echo "Checking out tag: ${params.VERSION}"
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "refs/tags/${params.VERSION}"]],
                            userRemoteConfigs: [[url: 'https://github.com/username/app.git']]
                        ])
                    } else {
                        echo "Checking out branch: ${params.VERSION}"

                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "${params.VERSION}"]],
                            userRemoteConfigs: [[
                                url: 'https://github.com/Sejal302002/slip8.git',
                                credentialsId: 'token1'
                            ]]
                        ])
                    }
                }

                echo "Repository checked out successfully"
                sh 'git log -1 --pretty=format:"Commit: %h - %an, %ar : %s"'
            }
        }

        stage('Build') {
            steps {
                echo '=========================================='
                echo 'Stage: Build'
                echo "Environment: ${params.DEPLOY_ENV}"
                echo "Version: ${params.VERSION}"
                echo '=========================================='

                sh 'node -v'
                sh 'npm -v'

                sh 'npm install'
                sh 'npm run build'

                echo 'Build completed successfully'
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS == true } }
            steps {
                echo '=========================================='
                echo 'Stage: Test'
                echo '=========================================='

                sh 'npm test'
            }
            post {
                always {
                    echo 'Test stage execution completed'
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                }
            }
        }

        stage('Deploy to Dev') {
            when { expression { params.DEPLOY_ENV == 'dev' } }
            steps {
                echo '=========================================='
                echo 'Deploying to Dev'
                echo '=========================================='

                sh '''
                    chmod +x scripts/deploy-dev.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-dev.sh
                '''
            }
        }

        stage('Deploy to Staging') {
            when { expression { params.DEPLOY_ENV == 'staging' } }
            steps {
                echo '=========================================='
                echo 'Deploying to Staging'
                echo '=========================================='

                sh '''
                    chmod +x scripts/deploy-staging.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-staging.sh
                '''
            }
        }

        stage('Deploy to Production') {
            when { expression { params.DEPLOY_ENV == 'prod' } }
            steps {
                echo '=========================================='
                echo 'Deploying to Production'
                echo '=========================================='

                sh '''
                    chmod +x scripts/deploy-prod.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-prod.sh
                '''
            }
        }
    }

    post {
        always {
            echo '=========================================='
            echo 'Pipeline Execution Summary'
            echo '=========================================='
            echo "RUN_TESTS: ${params.RUN_TESTS}"
            echo "DEPLOY_ENV: ${params.DEPLOY_ENV}"
            echo "VERSION: ${params.VERSION}"
            echo "Build Status: ${currentBuild.result ?: 'SUCCESS'}"
            echo '=========================================='
        }

        success {
            echo 'Pipeline executed successfully!'
            archiveArtifacts artifacts: '**/*', excludes: 'node_modules/**,coverage/**,.git/**'
        }

        failure {
            echo 'Pipeline failed!'
            echo 'Check console output for details'
        }
    }
}

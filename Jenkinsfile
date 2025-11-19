pipeline {
    agent any
    
    // Pipeline parameters
    parameters {
        // Boolean parameter: RUN_TESTS
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run unit tests before deployment'
        )
        
        // Choice parameter: DEPLOY_ENV
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'staging', 'prod'],
            description: 'Deployment environment'
        )
        
        // String parameter: VERSION
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
                    // Check if VERSION is a tag (starts with 'v') or branch
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
  branches: [[name: "main"]],
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
                
                sh 'npm install'
                sh 'npm run build'
                
                echo 'Build completed successfully'
            }
        }
        
        stage('Test') {
            // Conditional stage: Run only when RUN_TESTS is true
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo '=========================================='
                echo 'Stage: Test (RUN_TESTS = true)'
                echo '=========================================='
                
                sh 'npm test'
                
                echo 'Tests completed successfully'
            }
            post {
                always {
                    echo 'Test stage execution completed'
                    // Archive test results if needed
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                }
            }
        }
        
        stage('Deploy to Dev') {
            // Conditional stage: Deploy to dev environment
            when {
                expression { params.DEPLOY_ENV == 'dev' }
            }
            steps {
                echo '=========================================='
                echo 'Stage: Deploy to Development'
                echo "Environment: ${params.DEPLOY_ENV}"
                echo "Version: ${params.VERSION}"
                echo '=========================================='
                
                sh '''
                    chmod +x scripts/deploy-dev.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-dev.sh
                '''
                
                echo 'Deployment to development completed'
            }
        }
        
        stage('Deploy to Staging') {
            // Conditional stage: Deploy to staging environment
            when {
                expression { params.DEPLOY_ENV == 'staging' }
            }
            steps {
                echo '=========================================='
                echo 'Stage: Deploy to Staging'
                echo "Environment: ${params.DEPLOY_ENV}"
                echo "Version: ${params.VERSION}"
                echo '=========================================='
                
                sh '''
                    chmod +x scripts/deploy-staging.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-staging.sh
                '''
                
                echo 'Deployment to staging completed'
            }
        }
        
        stage('Deploy to Production') {
            // Conditional stage: Deploy to production environment
            when {
                expression { params.DEPLOY_ENV == 'prod' }
            }
            steps {
                echo '=========================================='
                echo 'Stage: Deploy to Production'
                echo "Environment: ${params.DEPLOY_ENV}"
                echo "Version: ${params.VERSION}"
                echo '=========================================='
                
                sh '''
                    chmod +x scripts/deploy-prod.sh
                    export VERSION="${params.VERSION}"
                    ./scripts/deploy-prod.sh
                '''
                
                echo 'Deployment to production completed'
            }
        }
    }
    
    post {
        always {
            echo '=========================================='
            echo 'Pipeline Execution Summary'
            echo '=========================================='
            echo "Parameters used:"
            echo "  RUN_TESTS: ${params.RUN_TESTS}"
            echo "  DEPLOY_ENV: ${params.DEPLOY_ENV}"
            echo "  VERSION: ${params.VERSION}"
            echo "Build Status: ${currentBuild.result ?: 'SUCCESS'}"
            echo "Build Number: ${env.BUILD_NUMBER}"
            echo "Build Duration: ${currentBuild.durationString}"
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


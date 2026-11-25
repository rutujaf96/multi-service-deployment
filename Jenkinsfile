pipeline {
    agent any

    parameters {
        choice(name: 'MICROSERVICE',
               choices: ['payment-service', 'customer-service',
                         'order-service', 'healthcare-service'],
               description: 'Select one microservice')
        choice(name: 'ENVIRONMENT',
               choices: ['dev', 'qa', 'uat', 'prod'],
               description: 'Select deployment environment')
        choice(name: 'ACTION',
               choices: ['BUILD', 'DEPLOY', 'RESTART', 'ROLLBACK'],
               description: 'Select operation')
    }

    environment {
        AWS_REGION = 'eu-north-1'
        // Set ECR_REGISTRY and EKS_CLUSTER using Jenkins configuration.
    }

    stages {
        stage('Validate') {
            steps {
                script {
                    if (!(params.MICROSERVICE in [
                        'payment-service', 'customer-service',
                        'order-service', 'healthcare-service'
                    ])) {
                        error('Invalid microservice')
                    }
                    if (!(params.ENVIRONMENT in ['dev', 'qa', 'uat', 'prod'])) {
                        error('Invalid environment')
                    }
                    echo "Service=${params.MICROSERVICE}, Environment=${params.ENVIRONMENT}, Action=${params.ACTION}"
                }
            }
        }

        stage('Production approval') {
            when {
                expression { params.ENVIRONMENT == 'prod' && params.ACTION != 'BUILD' }
            }
            steps {
                input message: "Approve ${params.ACTION} for ${params.MICROSERVICE} in PROD?",
                      ok: 'Approve'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and test') {
            when { expression { params.ACTION == 'BUILD' || params.ACTION == 'DEPLOY' } }
            steps {
                dir("services/${params.MICROSERVICE}") {
                    sh 'mvn -B clean test package'
                }
            }
        }

        stage('Docker build') {
            when { expression { params.ACTION == 'DEPLOY' } }
            steps {
                dir("services/${params.MICROSERVICE}") {
                    sh 'docker build -t ${MICROSERVICE}:${BUILD_NUMBER} .'
                }
                echo 'Add authenticated ECR login and image push here.'
            }
        }

        stage('Deploy / operate') {
            when { expression { params.ACTION != 'BUILD' } }
            steps {
                echo "Selected operation: ${params.ACTION}"
                echo "Environment config: environments/${params.ENVIRONMENT}/"
                echo 'Connect this stage to your approved EKS context and deployment scripts.'
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully.' }
        failure { echo 'Pipeline failed. Review Console Output.' }
        always  { echo "Result: ${currentBuild.currentResult}" }
    }
}

pipeline {
    agent any

    parameters {
        // UI मधील ४ ड्रॉपडाउन्सशी हे जुळलं पाहिजे
        string(name: 'APP_SERVICE', defaultValue: 'PAYMENTS', description: 'App Domain')
        string(name: 'MICROSERVICE', defaultValue: 'payment-service', description: 'Select one microservice')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'uat', 'prod'], description: 'Select deployment environment')
        choice(name: 'DEPLOYMENT_ACTION', choices: ['BUILD', 'DEPLOY', 'RESTART', 'ROLLBACK'], description: 'Select operation')
    }

    environment {
        AWS_REGION = 'eu-north-1'
        AWS_ACCOUNT_ID = 'YOUR_AWS_ACCOUNT_ID' // तुमचा AWS Account ID येथे टाका
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPO = "${ECR_REGISTRY}/${params.MICROSERVICE}"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Validate') {
            steps {
                script {
                    if (!params.MICROSERVICE?.trim()) {
                        error('Microservice is required')
                    }
                    echo "Service=${params.MICROSERVICE}, Env=${params.ENVIRONMENT}, Action=${params.DEPLOYMENT_ACTION}"
                }
            }
        }

        stage('Production approval') {
            when {
                expression { params.ENVIRONMENT == 'prod' && params.DEPLOYMENT_ACTION != 'BUILD' }
            }
            steps {
                // इथे पाइपलाइन थांबेल आणि तुमची मंजुरी (Approve) मागेल
                input message: "Approve ${params.DEPLOYMENT_ACTION} for ${params.MICROSERVICE} in PROD?", ok: 'Approve'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and test') {
            when { expression { params.DEPLOYMENT_ACTION == 'BUILD' || params.DEPLOYMENT_ACTION == 'DEPLOY' } }
            steps {
                dir("services/${params.MICROSERVICE}") {
                    sh 'mvn -B clean test package'
                }
            }
        }

        stage('Docker build & Push to ECR') {
            when { expression { params.DEPLOYMENT_ACTION == 'DEPLOY' } }
            steps {
                dir("services/${params.MICROSERVICE}") {
                    // १. Docker Image बनवणे
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                    
                    // २. ECR मध्ये लॉगिन करणे आणि पुश करणे
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy / Operate') {
            when { expression { params.DEPLOYMENT_ACTION != 'BUILD' } }
            steps {
                dir("services/${params.MICROSERVICE}") {
                    // ३. तुमच्या bash स्क्रिप्ट्स कॉल करणे (ज्या आपण आधी बनवल्या होत्या)
                    script {
                        if (params.DEPLOYMENT_ACTION == 'DEPLOY') {
                            sh "./deploy.sh ${params.ENVIRONMENT} ${ECR_REPO}:${IMAGE_TAG} ${AWS_REGION}"
                        } 
                        else if (params.DEPLOYMENT_ACTION == 'RESTART') {
                            sh "./restart.sh ${params.ENVIRONMENT} ${params.MICROSERVICE}"
                        } 
                        else if (params.DEPLOYMENT_ACTION == 'ROLLBACK') {
                            sh "./rollback.sh ${params.ENVIRONMENT} ${params.MICROSERVICE}"
                        }
                    }
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully.' }
        failure { echo 'Pipeline failed. Review Console Output.' }
        always  { echo "Result: ${currentBuild.currentResult}" }
    }
}

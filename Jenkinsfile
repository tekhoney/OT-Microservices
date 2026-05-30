@Library('my-shared-library') _

pipeline {
    agent any
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Select target environment')
        booleanParam(name: 'TRIGGER_ROLLBACK', defaultValue: false, description: 'Check to simulate a rollback')
    }

    environment {
        // Target only the attendance subdirectory 
        TARGET_SERVICE = 'attendance'
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    // Initialize the deployment manager class from your shared library
                    deployment = new org.company.DeploymentManager(this, params.DEPLOY_ENV, env.TARGET_SERVICE)
                }
            }
        }

        stage('Validate Target') {
            steps {
                script {
                    deployment.validate()
                }
            }
        }

        stage('Deploy Microservice') {
            steps {
                script {
                    if (params.TRIGGER_ROLLBACK) {
                        error "Simulated pipeline failure during attendance deployment."
                    } else {
                        deployment.deploy()
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                if (deployment != null) {
                    deployment.rollback()
                }
            }
        }
    }
}

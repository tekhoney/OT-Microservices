@Library('my-shared-library') _

import org.company.DeploymentManager

pipeline {
    agent any
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Select target environment')
    }

    stages {
        stage('Checkout & Build') {
            steps {
                // Simulating working inside the OT-Microservices attendance app context
                git url: 'https://github.com/tekhoney/OT-Microservices.git', branch: 'master'
                echo "Building Attendance Microservice..."
                // Example: sh './mvnw clean package' or docker build commands here
            }
        }
        
        stage('Execute Library Logic') {
            steps {
                script {
                    // Instantiate the custom class object, passing 'this' context and the environment parameter
                    def deployer = new DeploymentManager(this, params.DEPLOY_ENV)
                    
                    try {
                        // 1. Run Validation
                        deployer.validate()
                        
                        // 2. Run Environment-Specific Deployment
                        deployer.deploy()
                        
                    } catch (Exception e) {
                        echo "Deployment encountered an error: ${e.message}"
                        // 3. Trigger the Recreate Rollback Strategy if deployment crashes
                        deployer.rollback()
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
}

// Jenkinsfile  — root of tekhoney/OT-Microservices
@Library('ot-shared-lib') _                  // ← loads jenkins-shared-library

import org.company.DeploymentManager         // ← imports the Groovy class

pipeline {
    agent any

    // ── Build parameters — choose env + service at runtime ─────────────────
    parameters {
        choice(
            name        : 'DEPLOY_ENV',
            choices     : ['dev', 'staging', 'prod'],
            description : 'Target environment'
        )
        choice(
            name        : 'SERVICE_NAME',
            choices     : ['employee', 'attendance', 'salary', 'notification', 'frontend'],
            description : 'OT-Microservices service to deploy'
        )
        string(
            name        : 'IMAGE_TAG',
            defaultValue: '',
            description : 'Image tag to deploy (leave blank to use env default: dev→latest, staging→rc, prod→stable)'
        )
        string(
            name        : 'ROLLBACK_TAG',
            defaultValue: '',
            description : 'Known-good image tag for rollback (leave blank = env default tag)'
        )
    }

    environment {
        // Makes the selected values available as env vars throughout the pipeline
        TARGET_ENV  = "${params.DEPLOY_ENV}"
        SVC_NAME    = "${params.SERVICE_NAME}"
        IMG_TAG     = "${params.IMAGE_TAG}"
        RB_TAG      = "${params.ROLLBACK_TAG}"
    }

    stages {

        // ── Stage 1: Checkout ───────────────────────────────────────────────
        stage('Checkout') {
            steps {
                echo "Checking out OT-Microservices..."
                checkout scm
                echo "✔ Checkout complete"
            }
        }

        // ── Stage 2: Validate only (no kubectl) ─────────────────────────────
        stage('Validate Environment') {
            steps {
                script {
                    def manager = new DeploymentManager(this, env.TARGET_ENV)
                    // validate() will ask for manual approval if TARGET_ENV == prod
                    manager.validate()
                }
            }
        }

        // ── Stage 3: Deploy to DEV ───────────────────────────────────────────
        stage('Deploy to Dev') {
            when {
                expression { env.TARGET_ENV == 'dev' }
            }
            steps {
                script {
                    echo "=== Deploying ${env.SVC_NAME} to DEV ==="
                    def manager = new DeploymentManager(this, 'dev')
                    manager.validate()
                    manager.deploy(env.SVC_NAME, env.IMG_TAG ?: null)
                }
            }
            post {
                failure {
                    script {
                        echo "=== DEV deployment failed — Recreate Rollback triggered ==="
                        def manager = new DeploymentManager(this, 'dev')
                        manager.rollback(env.SVC_NAME, env.RB_TAG ?: null)
                    }
                }
            }
        }

        // ── Stage 4: Deploy to Staging ──────────────────────────────────────
        stage('Deploy to Staging') {
            when {
                expression { env.TARGET_ENV == 'staging' }
            }
            steps {
                script {
                    echo "=== Deploying ${env.SVC_NAME} to STAGING ==="
                    def manager = new DeploymentManager(this, 'staging')
                    manager.validate()
                    manager.deploy(env.SVC_NAME, env.IMG_TAG ?: null)
                }
            }
            post {
                failure {
                    script {
                        echo "=== STAGING deployment failed — Recreate Rollback triggered ==="
                        def manager = new DeploymentManager(this, 'staging')
                        manager.rollback(env.SVC_NAME, env.RB_TAG ?: null)
                    }
                }
            }
        }

        // ── Stage 5: Deploy to Prod ──────────────────────────────────────────
        stage('Deploy to Prod') {
            when {
                expression { env.TARGET_ENV == 'prod' }
            }
            steps {
                script {
                    echo "=== Deploying ${env.SVC_NAME} to PRODUCTION ==="
                    // validate() already asked for approval above — deploy directly
                    def manager = new DeploymentManager(this, 'prod')
                    manager.deploy(env.SVC_NAME, env.IMG_TAG ?: null)
                }
            }
            post {
                failure {
                    script {
                        echo "=== PROD deployment failed — Recreate Rollback triggered ==="
                        def manager = new DeploymentManager(this, 'prod')
                        manager.rollback(env.SVC_NAME, env.RB_TAG ?: null)
                    }
                }
            }
        }

        // ── Stage 6: Using the vars/ helper (alternate approach demo) ────────
        // This stage shows @Library usage via the global vars/ step directly
        stage('Verify via deployOTService step') {
            when {
                expression { env.TARGET_ENV == 'dev' }
            }
            steps {
                echo "Demo: calling global vars step directly..."
                deployOTService(
                    environment : 'dev',
                    serviceName : env.SVC_NAME,
                    imageTag    : env.IMG_TAG ?: null
                )
            }
        }
    }

    // ── Post pipeline notifications ──────────────────────────────────────────
    post {
        success {
            echo "✅ Pipeline SUCCESS: ${env.SVC_NAME} → ${env.TARGET_ENV}"
        }
        failure {
            echo "❌ Pipeline FAILED: ${env.SVC_NAME} → ${env.TARGET_ENV}"
        }
        always {
            echo "Pipeline completed for env=${env.TARGET_ENV}, service=${env.SVC_NAME}"
        }
    }
}

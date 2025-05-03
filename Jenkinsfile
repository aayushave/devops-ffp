@Library('shared') _

pipeline {
    agent any
    
    environment {
        // Update the main app image name to match the deployment file
        DOCKER_IMAGE_NAME = 'riviwa6166/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'riviwa6166/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GIT_REPO ="https://github.com/aayushave/devops-ffp.git"
        // GIT_BRANCH = "main"
        GIT_BRANCH = "${env.BRANCH_NAME}"

    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }

        stage('Print Branch') {
            steps {
                // echo "Running CI for branch: ${CURRENT_BRANCH}"
                 sh 'printenv'
            }
        }
        
        stage('Conditional Step') {
            when {
                branch 'main'
            }
            steps {
                echo "This step runs only on the main branch."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME.startsWith("release-")) {
                        echo "Running deployment logic for a release branch: ${env.BRANCH_NAME}"

                    } else if(env.BRANCH_NAME.startsWith("Feature-")){
                            echo "Running deployment logic for a Feature branch: ${env.BRANCH_NAME}"
                        // Deployment commands here
                    } else {
                        echo "Skipping deployment for non-release branch"
                    }
                }
            }
        }
        stage('Clone Repository') {
            steps {
                script {
                    clone(env.GIT_REPO,env.GIT_BRANCH)
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    // Create directory for results
                  
                    trivy_scan()
                    
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
                
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }
        
        // Add this new stage
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'rr.ayushr@gmail.com'
                    )
                }
            }
        }
    }
}

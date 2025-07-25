pipeline {
    agent any

    environment {
        REGISTRY = "minnathdhani"
        DOCKER_BUILDKIT = "0"
        FRONTEND_IMAGE = "${REGISTRY}/mern-frontend"
        BACKEND_IMAGE = "${REGISTRY}/mern-backend"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/minnathdhani/MERN-Assignment.git'
                script {
                    sh 'git rev-parse HEAD > .repo_commit'
                    def changed = false

                    if (fileExists('.repo_commit.old')) {
                        def oldCommit = readFile('.repo_commit.old').trim()
                        def newCommit = readFile('.repo_commit').trim()
                        if (oldCommit != newCommit) {
                            echo "Codebase has changed."
                            changed = true
                        }
                    } else {
                        changed = true
                    }

                    if (!changed) {
                        echo "No changes detected. Skipping build."
                        currentBuild.result = 'SUCCESS'
                        error('Build skipped.')
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building frontend Docker image from ./learnerReportCS_frontend"
                sh "docker build -t ${FRONTEND_IMAGE}:latest ./learnerReportCS_frontend"

                echo "Building backend Docker image from ./learnerReportCS_backend"
                sh "docker build -t ${BACKEND_IMAGE}:latest ./learnerReportCS_backend"
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker push ${FRONTEND_IMAGE}:latest
                        docker push ${BACKEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                sh """
                    helm upgrade --install mern-app ./mern-chart \
                      --namespace mern \
                      --create-namespace \
                      --values ./mern-chart/values.yaml
                """
            }
        }
    }

    post {
        success {
            sh 'cp .repo_commit .repo_commit.old || true'
        }
        always {
            cleanWs()
        }
    }
}

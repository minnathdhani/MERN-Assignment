pipeline {
    agent any

    environment {
        DOCKER_IMAGE_FE = 'minnathdhani/frontend-image'
        DOCKER_IMAGE_BE = 'minnathdhani/backend-image'
    }

    stages {
        stage('Clone Repos') {
            steps {
                git url: 'https://github.com/minnathdhani/MERN-Assignment.git', branch: 'main'
                // do the same for backend in a separate workspace
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_FE ./frontend'
                sh 'docker build -t $DOCKER_IMAGE_BE ./backend'
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE_FE'
                    sh 'docker push $DOCKER_IMAGE_BE'
                }
            }
        }

        stage('Deploy with HELM') {
            steps {
                sh 'helm upgrade --install mern-release ./helm-chart --namespace mern --create-namespace'
            }
        }
    }
}

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('build jar') {
            steps {
                script {
                    echo 'building the application...'
                 }
            }
        }

        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                }
            }
        }

        stage('deploy') {
            environment {
                    AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                    AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
            }
            steps {
                script {
                    echo 'deploying the application...'
                    kubectl create deployment nginx-deployment --image nginx
                }
            }
        }
    }
}

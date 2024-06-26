pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                echo 'Cloning the Project from Github to Jenkins Workspace'
                git branch: 'main', url: 'https://github.com/er-sanjaysah/preview.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                dir('src'){
                    sh 'ls'
                    sh "pwd"
                    echo 'Building docker image'
                    sh "docker build -t ssah6694/devops:${env.BUILD_NUMBER} ."
                }
            }
        }
        
        stage("Push Images"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh 'echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh "docker push ssah6694/devops:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Update Manifest') {
            steps {
                echo "Triggering update manifest job"
                build job: 'argocd-update-manifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)], propagate: true
            }
        }
    }
}

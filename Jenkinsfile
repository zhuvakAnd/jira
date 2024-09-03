pipeline {
    agent { label 'build-vm' }

    environment {
        GIT_KEY = credentials('github-key')
        REPO_OWNER = 'zhuvakAnd'
        REPO_NAME = 'jira_clone'
        RELEASE_VERSION = "v1.${env.BUILD_NUMBER}.0"
        RELEASE_NOTES = "Release notes for version 1.${env.BUILD_NUMBER}.0"
        BASE_PATH = '/home/andrii/jira_clone'
    }
    
    stages {       
        stage('Pull') {
            steps {
                dir("${BASE_PATH}"){
                    sh "git pull https://github.com/${REPO_OWNER}/${REPO_NAME}"
                }
            }
        }
        stage('Build frontend') {
            steps {
                sh "docker build ${BASE_PATH}/client -t nginx-image"
            }
        }
        stage('Build backend') {
            steps {
                sh "docker build ${BASE_PATH}/api -t node-image"
            }
        }
        stage('Run DB') {
            steps {
                sh "docker run --name psql -d -p 5432:5432 zhuvakandrii/jira:pg"
            }
        }
        stage('Run backend') {
            steps {
                sh "docker run --name node -d -p 3000:3000 node-image"
            }
        }
        stage('Run frontend') {
            steps {
                sh "docker run --name nginx -d -p 8080:80 nginx-image"
            }
        }
    }
}
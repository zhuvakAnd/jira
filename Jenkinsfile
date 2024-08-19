pipeline {
    agent { label 'build-vm' }
    
    environment {
        GIT_KEY = credentials('github-key')
        REPO_OWNER = 'zhuvakAnd'
        REPO_NAME = 'jira_clone'
        RELEASE_VERSION = "v1.${env.BUILD_NUMBER}.0"
        RELEASE_NOTES = "Release notes for version 1.${env.BUILD_NUMBER}.0"
        BASE_PATH = '/home/jenkins/jira_clone'
    }

    stages {       
        stage('Pull') {
            steps {
                dir("${BASE_PATH}"){
                    sh "git pull https://github.com/${REPO_OWNER}/${REPO_NAME}"
                }
            }
        }
        stage('Install packages') {
            steps {
                dir("${BASE_PATH}"){
                    sh 'npm run install-dependencies'
                }
            }
        }
        stage('Build') {
            steps {
                dir("${BASE_PATH}") {
                    sh 'npm run build'
                }
            }
        }

        stage('Start app') {
            steps {
                dir("${BASE_PATH}") {
                    sh 'npm run start:production'
                }
            }
        }

        stage('Check api connection') {
            steps {             
                sh 'curl localhost:3000'    
            }
        }

        stage('Check client connection') {
            steps {                
                sh 'curl localhost:8080'            
            }
        }

    
        stage('Test') {
            steps {
                dir("${BASE_PATH}/client") {
                    sh 'npm run test:cypress'
                }
            }
        }

        stage('Zip-front') {
            steps {
                dir("${BASE_PATH}/client") {
                    sh 'zip -r build.zip build'
                }
            }
        }
        stage('Zip-back') {
            steps {
                dir("${BASE_PATH}") {
                    sh 'zip -r buildback.zip api -x "api/node_modules/*" -x "api/src/*"'
                }
            }
        }
        
        stage('Create GitHub Release') {
            steps {
                script {
                    def createReleaseCommand = """
                    curl -H "Authorization: Bearer ${GIT_KEY}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -X POST \
                        -d '{ "tag_name": "${RELEASE_VERSION}", "name": "${RELEASE_VERSION}", "body": "${RELEASE_NOTES}", "draft": false, "prerelease": false }' \
                        https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/releases
                    """
                    sh createReleaseCommand
                }
            }
        }
        
        stage('Get upload url') {
            steps {
                script {
                    def releaseResponse = sh(script: """
                    curl -H "Authorization: Bearer ${GIT_KEY}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -X GET \
                        https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/releases/tags/${RELEASE_VERSION}
                    """, returnStdout: true).trim()
                    
                    def jsonSlurper = new groovy.json.JsonSlurper()
                    def jsonResponse = jsonSlurper.parseText(releaseResponse)
                    
                    if (jsonResponse.upload_url) {
                        UPLOAD_URL = jsonResponse.upload_url.replace("{?name,label}", "")
                        println "Cleaned upload URL: ${UPLOAD_URL}"
                    } else {
                        println "Error: 'upload_url' not found in the JSON response"
                    }
                }
            }
        }

        stage('Upload Front-end Artifact') {
            steps {
                script {
                    def buildArtifactPath = "${BASE_PATH}/client/build.zip"
                    
                    sh """
                    curl -H "Authorization: Bearer ${GIT_KEY}" \
                        -H "Content-Type: application/zip" \
                        -X POST \
                        --data-binary @"${buildArtifactPath}" \
                        "${UPLOAD_URL}?name=build.zip"
                    """
                }
            }
        }
        stage('Upload Back-end Artifact') {
            steps {
                script {
                    def buildArtifactPath = "${BASE_PATH}/buildback.zip"
                    
                    sh """
                    curl -H "Authorization: Bearer ${GIT_KEY}" \
                        -H "Content-Type: application/zip" \
                        -X POST \
                        --data-binary @"${buildArtifactPath}" \
                        "${UPLOAD_URL}?name=buildback.zip"
                    """
                }
            }
        }
    }
    post {
        always {
            sh "pm2 delete all"
        }
    }
}

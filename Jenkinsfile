pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME            = "${DOCKERHUB_CREDENTIALS_USR}/abdulahad9049/flask-todo-app-helm-argocd"  
    }

    stages {

        stage('Clone') {
            steps {
                git url: 'https://github.com/Ahad9049/flask-todo-app.git',  
                    branch: 'main',                                     
                    credentialsId: 'github-credentials'
            }
        }

        stage('Version') {
            steps {
                script {
                    sh 'git fetch --tags --force'

                    def tag = sh(
                        script: "git describe --tags --match 'v[0-9]*' --abbrev=0 2>/dev/null || echo 'v0.0.0'",
                        returnStdout: true
                    ).trim()

                    def isExact = sh(
                        script: "git describe --exact-match --tags HEAD 2>/dev/null && echo yes || echo no",
                        returnStdout: true
                    ).trim()

                    def version = tag.replaceFirst(/^v/, '')
                    if (isExact == 'no') {
                        def sha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                        version = "${version}-${BUILD_NUMBER}-${sha}"
                    }

                    env.VERSION = version
                    echo "Building version: ${env.VERSION}"
                }
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${VERSION} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push') {
            steps {
                sh 'echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin'
                sh "docker push ${IMAGE_NAME}:${VERSION}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            sh "docker rmi ${IMAGE_NAME}:${VERSION} ${IMAGE_NAME}:latest || true"
            sh 'docker logout'
        }
        success { echo "✅ Pushed ${IMAGE_NAME}:${VERSION}" }
        failure { echo "❌ Build failed" }
    }
}
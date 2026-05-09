pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME            = "abdulahad9049/flask-todo-app-helm-argocd"
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

            def latestTag = sh(
                script: "git tag --sort=-v:refname | grep -E '^v[0-9]+\\.[0-9]+\\.[0-9]+\$' | head -1",
                returnStdout: true
            ).trim()

            // If no tags exist start from v1.0.0
            if (!latestTag) {
                latestTag = 'v1.0.0'
            }

            def parts = latestTag.replaceFirst(/^v/, '').tokenize('.')
            def major = parts[0].toInteger()
            def minor = parts[1].toInteger()
            def patch = parts[2].toInteger() + 1

            env.VERSION = "v${major}.${minor}.${patch}"
            echo "Version: ${env.VERSION}"
        }
    }
}
        stage('Build') {
            steps {
                sh 'docker build -t ' + env.IMAGE_NAME + ':' + env.VERSION + ' -t ' + env.IMAGE_NAME + ':latest .'
            }
        }

        stage('Push') {
            steps {
                sh 'echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin'
                sh 'docker push ' + env.IMAGE_NAME + ':' + env.VERSION
                sh 'docker push ' + env.IMAGE_NAME + ':latest'
            }
        }
    }

    post {
        always {
            sh 'docker rmi ' + env.IMAGE_NAME + ':' + env.VERSION + ' ' + env.IMAGE_NAME + ':latest || true'
            sh 'docker logout'
        }
        success { echo "✅ Pushed ${IMAGE_NAME}:${VERSION}" }
        failure { echo "❌ Build failed" }
    }
}
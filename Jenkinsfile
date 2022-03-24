pipeline {
    agent any
    environment {
        DOCKER_PASSWORD = credentials("docker_password")
        GITHUB_TOKEN = credentials("github_token")
    }

    stages {
        stage('Build & Test') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Publish artefact') {
           steps {
                script {
                    sh([script: 'git fetch --tag', returnStdout: true]).trim()
                    env.MAJOR_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 1', returnStdout: true]).trim()
                    env.MINOR_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 2', returnStdout: true]).trim()
                    env.PATCH_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 3', returnStdout: true]).trim()
                    env.IMAGE_TAG = "${env.MAJOR_VERSION}.\$((${env.MINOR_VERSION} + 1)).${env.PATCH_VERSION}"
                    echo "${env.IMAGE_TAG}"
                }
                sh "docker build -t alinaeftn/hello-img:${env.IMAGE_TAG} ."
                sh "docker login docker.io -u alinaeftn -p ${DOCKER_PASSWORD}"
                sh "docker push alinaeftn/hello-img:${env.IMAGE_TAG}"
                sh "git tag ${env.IMAGE_TAG}"
                sh "git push https://$GITHUB_TOKEN@github.com/alinaeftn/service.git ${env.IMAGE_TAG}"
           }
        }

        stage ('Deploy and test') {
          steps {
            sh "printenv"
            sh 'docker-compose up -d hello'
          }
        }

    }
}

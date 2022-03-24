pipeline {
    agent any
    environment {
        DOCKER_PASSWORD = credentials("docker_password")
        GITHUB_TOKEN = credentials("github_token")
        IMAGE_TAG = "v1.5.0"
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
                MAJOR_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 1', returnStdout: true]).trim()
                MINOR_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 2', returnStdout: true]).trim()
                PATCH_VERSION = sh([script: 'git tag | tail -1 | cut -d . -f 3', returnStdout: true]).trim()
              }
                sh "docker build -t alinaeftn/hello-img:${MAJOR_VERSION}.\$((${MINOR_VERSION} + 1)).${PATCH_VERSION} ."
                sh "docker login docker.io -u alinaeftn -p ${DOCKER_PASSWORD}"
                sh "docker push alinaeftn/hello-img:${MAJOR_VERSION}.\$((${MINOR_VERSION} + 1)).${PATCH_VERSION}"
                sh "git tag ${MAJOR_VERSION}.\$((${MINOR_VERSION} + 1)).${PATCH_VERSION}"
                sh "git push https://$GITHUB_TOKEN@github.com/alinaeftn/service.git ${MAJOR_VERSION}.\$((${MINOR_VERSION} + 1)).${PATCH_VERSION}"
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

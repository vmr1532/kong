pipeline {
    agent none
    environment{
        UPDATE_CACHE = "true"
        DOCKER_CREDENTIALS = credentials('dockerhub')
        DOCKER_USERNAME = "${env.DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD = "${env.DOCKER_CREDENTIALS_PSW}"
        KONG_BUILD_TOOLS = "origin/feat/kong-jenkins"
    }
    stages {
        stage('Build Kong') {
            agent {
                node {
                    label 'docker-compose'
                }
            }
            environment {
                KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
            }
            steps {
                sh 'make setup-kong-build-tools'
                sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                dir('../kong-build-tools') { sh 'make kong-test-container' }
            }
        }
        stage('Nightly Releases') {
            parallel {
                stage('Ubuntu Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'deb'
                        RESTY_IMAGE_BASE = 'ubuntu'
                        RESTY_IMAGE_TAG = 'xenial'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        AWS_ACCESS_KEY = credentials('AWS_ACCESS_KEY')
                        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
                        DOCKER_MACHINE_ARM64_NAME = "jenkins-kong-${env.BUILD_NUMBER}"
                        REPOSITORY_NAME = 'kong-private-nightly'
                        KONG_PACKAGE_NAME = 'kong'
                        REPOSITORY_OS_NAME = "${env.BRANCH_NAME}"
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=trusty BUILDX=false make nightly-release'
                    }
                }
            }
        }
    }
}

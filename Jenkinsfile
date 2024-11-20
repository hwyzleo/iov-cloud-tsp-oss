pipeline {
    agent any

    environment {
        REPO_URL = "http://${env.MAVEN_URL}/repository/maven-snapshots/"
        REPO_ID = "snapshots"
        PROJECT_NAME = "${env.JOB_NAME}"
        IMAGE_NAME = "${env.REGISTRY_URL}/${PROJECT_NAME}:${env.BUILD_NUMBER}"
    }

    tools {
        maven 'M3'
    }

    stages {
        stage('构建并发布') {
            steps {
                script {
                    sh "mvn clean deploy -DaltDeploymentRepository=${REPO_ID}::default::${REPO_URL}"
                    dir('oss-api') {
                        sh "mvn clean deploy -DaltDeploymentRepository=${REPO_ID}::default::${REPO_URL}"
                    }
                }
            }
        }
        stage('构建镜像') {
            steps {
                script {
                    dir('oss-service') {
                        sh "docker build -t ${IMAGE_NAME} -f ../Dockerfile ."
                    }
                }
            }
        }
        stage('上传镜像') {
            steps {
                sh "docker push ${IMAGE_NAME}"
            }
        }
        stage('运行镜像') {
            steps {
                sh "docker stop ${PROJECT_NAME}"
                sh "docker rm ${PROJECT_NAME}"
                sh "docker pull ${IMAGE_NAME}"
                sh "docker run -d --name ${PROJECT_NAME} ${IMAGE_NAME}"
                sh "sleep 10"
                sh "docker logs ${PROJECT_NAME}"
            }
        }
    }
}
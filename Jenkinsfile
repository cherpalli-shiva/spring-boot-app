pipeline {
    agent any {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock // mount the Docker socket'
        }  
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                //build the project and create a jar file
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = 'http://http://3.17.205.43:9000'
            }
            steps {
                withCredentials([string(credentialsId:'sonnarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        stage('Build and push docker image') {
            environment {
                DOCKER_IMAGE = 'abhishekf5/ultimatecicd:${BUILD_NUMBER}'
                //Dockerfile_location = "spring-boot-app/Dockerfile"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Upate Deployment file') {
            environment {
                GIT_REPO_NAME = 'spring-boot-app'
                GIT_USER_NAME = 'cherpalli'
            }
            steps {
                withCredentials([string(credentials:'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config user.email "cherpallishiva123@gmail.com'
                    git config user.name "${GIT_USER_NAME}"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" /spring-boot-app/K8S/deployment.yaml
                    git add /spring-boot-app/K8S/deployment.yaml
                    git commit -m "Updated deployment file with build number ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        
        }

    }

}

pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }

    environment {
        DOCKER_IMAGE = 'borhen05/jenkins-tp'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/borhen10/jenkins-tp-atelier2.git'
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                    java -version
                    mvn -version
                '''
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean compile -U'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(
                    credentialsId: 'sonar-token',
                    variable: 'SONAR_TOKEN'
                )]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.host.url=http://10.0.2.15:9000 \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline SUCCESS 🚀'
        }
        failure {
            echo 'Pipeline FAILED ❌'
        }
    }
}
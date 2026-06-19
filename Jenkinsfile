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
        sh 'mvn clean package -DskipTests -U'
    }
}
        stage('SonarQube Analysis') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    withCredentials([string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.host.url=http://192.168.49.2:32000 \
                            -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
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
        stage('Kubernetes Deploy') {
            steps {
                sh """
                    sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' k8s/spring-deployment.yaml
                    kubectl apply -f k8s/mysql-deployment.yaml -n devops
                    kubectl apply -f k8s/spring-deployment.yaml -n devops
                    kubectl rollout status deployment/studentmang-app -n devops --timeout=120s
                """
            }
        }
    }
    post {
        success {
            echo 'Pipeline SUCCESS 🚀'
            sh 'kubectl get pods -n devops'
        }
        failure {
            echo 'Pipeline FAILED ❌'
            sh 'kubectl get pods -n devops'
        }
    }
}
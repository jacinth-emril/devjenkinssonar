pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = '172.31.2.181:8083'
        IMAGE_NAME = '172.31.2.181:8083/devjenkins:1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:jacinth-emril/devjenkinssonar.git'
                )
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      aquasec/trivy:latest \
                      image \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      $IMAGE_NAME
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-docker',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASS" | docker login $DOCKER_REGISTRY \
                        -u "$NEXUS_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push to Nexus') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage in Console Output.'
        }
    }
}

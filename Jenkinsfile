pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:jacinth-emril/devjenkinssonar.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([string(credentialsId: 'jenkinssonar', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                            -Dsonar.projectKey=devjenkins \
                            -Dsonar.projectName=devjenkins
                            -Dsonar.token=$SONAR_TOKEN
                        ''' 
                }
            }
        }
    }

    post {
        success {
            echo 'Build and SonarQube analysis completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the Console Output.'
        }
    }
}

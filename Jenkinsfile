pipeline {

  environment {
    registry = "ak15023/train-schedule-app"
    registryCredential = 'docker_hub_login'
    dockerImage = ''
    }
    agent any
    stages {

        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        // stage('Cloning Git') {
        //     steps {
        //         git 'https://github.com/gustavoapolinario/microservices-node-example-todo-frontend.git'
        //     }
        // }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                    }
                }
            }
        }
    }
}
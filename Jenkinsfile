pipeline {
    agent any
    stages {

        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Building Docker image') {
            steps{
                script {
                    dockerImage = docker.build("ak15023/train-schedule-app")
                    deckerImage.inside{ sh 'echo $(curl localhost:8080)'}
                }
            }
        }
        stage('Push Docker Image') {
            steps{
                script {
                    docker.withRegistry('https://registry.hub.docker.com', docker_hub_login ) {
                    dockerImage.push("${env.BUIILD_ID}")
                    dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Production') {

            when{
                branch 'master'
            }
            steps{
                input "Deploy to Production?"
                milestone(1)

                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){

                    sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \" docker pull ak15023/train-schedule-app:${env.BUIILD_ID}\""
                    try{
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \" docker stop ak15023/train-schedule-app\""
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \" docker rm ak15023/train-schedule-app\""
                    }
                    catch (err){
                        echo: 'caught error: $err'
                    }
                    sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker run --restart=always --name train-schedule-app :wq!-p 8080:8080 -d ak15023/train-schedule-app:${env.BUIILD_ID}\""

                }
            }
        }
    }
}
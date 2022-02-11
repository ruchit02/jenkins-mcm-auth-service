def app
pipeline {
    agent any

    tools {
        
        maven 'maven-3-on-docker-cont'
    }

    stages {
        
        stage('Checkout SCM') {
            steps {

                git branch: 'main', url: 'https://github.com/ruchit02/jenkins-auth-service.git'
                echo 'Repository has been cloned into the current workspace'
            }
        }

        stage('Create a JAR file of the source code') {
            steps {

                sh 'mvn clean package'
                echo 'JAR file created successfully'
            }
        }

        stage('Build the docker image using the dockerfile in the root folder') {
            steps {

                script {
                    app = docker.build("t0pn0tch/auth-image")
                }

                echo 'docker image built successfully'
            }
        }

        stage('Push docker image to dockerhub') {
            steps {

                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-topnotch') {
                        app.push("latest")
                    }
                }

                echo 'Image successfully pushed to dockerhub'
            }
        }

        stage('Deploy Pod in local kubernetes cluster') {
            steps {

                withKubeConfig([credentialsId: 'minikube-kubeconfig-file', serverUrl: 'https://192.168.99.102:8443']) {

                    sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.22.1/bin/linux/amd64/kubectl"'  
                    sh 'chmod u+x ./kubectl'
                    sh './kubectl apply -f authentication.yml'
                }

                echo 'Pod successfully deployed in kubernetes cluster'
            }
        }

        stage('Send Notification') {
            steps {

                mail bcc: '',
                        body: 'Hey! Someone just deployed a pod in your kubernetes cluster through jenkins! Kindly checkout whether its an authorized user',
                        cc: 'swethupaturu@gmail.com',
                        from: '',
                        replyTo: 'noreply@gmail.com',
                        subject: 'Pod Deploy Notification',
                        to: 'darjiruchit02@gmail.com'

                echo 'Notification sent to respectful owners'
            }
        }
    }
}
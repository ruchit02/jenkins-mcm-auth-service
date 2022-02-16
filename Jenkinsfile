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
				echo "HEY CHECK THIS OUT, THIS IS THE BUILD-ID ${env.BUILD_ID}"
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
                        app.push("${env.BUILD_ID}")
                    }
                }

                echo 'Image successfully pushed to dockerhub'
            }
        }
		
		stage('Execute shell script') {
			steps {
				
				script {
					sh '''
						rm -rf mcm-topnotch-new
						git clone https://github.com/ruchit02/mcm-topnotch-new.git
						cd mcm-topnotch-new/
						git init
						git config --global user.email "darjiruchit02@gmail.com"
						git config --global user.name "Ruchit Darji"
						git branch -M main
						sed -i "s|image: t0pn0tch/auth-image.*|image: t0pn0tch/auth-image:$BUILD_ID|" everything/auth-deploy.yaml
						git add .
						git commit -m "Initial launch"
						git remote remove origin
						git remote add origin $MCM_APP_TOPNOTCH_GITHUB_REPO
						git push -u origin main
					'''
				}
			}
		}

        stage('Send Notification') {
            steps {

                mail bcc: '',
                        body: 'Hey! Someone just deployed a pod in your kubernetes cluster through jenkins! Kindly checkout whether its an authorized user',
                        cc: 'darjiruchit02@gmail.com',
                        from: '',
                        replyTo: 'noreply@gmail.com',
                        subject: 'Pod Deploy Notification',
                        to: 'darjiruchit02@gmail.com'

                echo 'Notification sent to respectful owners'
            }
        }
    }
}
pipeline {
	environment {
		IMAGE_NAME = "alpinehelloworld"
		IMAGE_TAG = "latest"
		STAGIN = "samirerr-staging"
		PRODUCTION = "samirerr-production"
		}
	agent none
	stages {
		stage('builder l image') {
			agent any
			steps {
				script {
					sh '''
						docker build -t samirerr/$IMAGE_NAME:$IMAGE_TAG .'
						'''
					}
				}
				
			}
		
		stage('demarrer contenaire basé sur l image buildee') {
			agent any
			steps {
				script {
					sh '''
						docker run --name $IMAGE_NAME -d -p 80:5000 -e port=5000 samirerr/$IMAGE_NAME:$IMAGE_TAG'
						sleep 5
						'''
					}
				}
				
			}
			
		stage('tester image') {
			agent any
			steps {
				script {
					sh '''
						curl http://localhost | grep -q "Hello world!"
						'''
					}
				}
				
			}
		stage('nettoyer le container') {
			agent any
			steps {
				script {
					sh '''
						docker stop $IMAGE_NAME
						docker rm $IMAGE_NAME
						'''
					}
				}
				
			}
		
		stage('pusher l image dans le staging et la deployer') {
			when {
				expression { GIT_BRANCH == 'origin/master'}
			}
			agent any
			environment { 
				HEROKU_API_KEY= credentials('heroku_api_key')
			}
			steps {
				script {
					sh '''
						heroku container:login
						heroku create $STAGING || echo "project existe déja"
						heroku container:push -a $STAGING web
						heroku container:release -a $STAGING web
						'''
						}
				}
		}
	}
}

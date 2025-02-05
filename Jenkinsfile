pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       STAGING = "samirerr-staging"
       PRODUCTION = "samirerr-production"
     }
     agent none
     stages {
         stage('Builder  l image') {
             agent any
             steps {
                script {
                  sh 'docker build -t samirerr/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Demarrer l image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 samirerr/$IMAGE_NAME:$IMAGE_TAG 
                    sleep 5
                 '''
               }
            }
       }
       stage('Tester l image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://localhost | grep -q "Hello world!"
                '''
              }
           }
      }
      stage('nettoyer le Container') {
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
     stage('Pusher l image dans le stagging et la deployer') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku')
      }  
      steps {
          script {
            sh '''
              /var/lib/snapd/snap/bin/heroku container:login
              /var/lib/snapd/snap/bin/heroku create $STAGING || echo "project already exist"
              /var/lib/snapd/snap/bin/heroku container:push -a $STAGING web
              /var/lib/snapd/snap/bin/heroku container:release -a $STAGING web
            '''
          }
        }
     }
     stage('Pusher l image dans la Prod et la deployer') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku')
      }  
      steps {
          script {
            sh '''
              /var/lib/snapd/snap/bin/heroku container:login
              /var/lib/snapd/snap/bin/heroku create $PRODUCTION || echo "project already exist"
              /var/lib/snapd/snap/bin/heroku container:push -a $PRODUCTION web
              /var/lib/snapd/snap/bin/heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
  }
     
      post {
       always {
			   script {
				 slackNotifier currentBuild.result
			 }
		}  
    }
}

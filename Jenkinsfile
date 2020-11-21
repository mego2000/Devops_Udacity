pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker build -t mohamedghonim/capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD 
						sudo docker push mohamedghonim/capstone 
					'''
				}
			}
		}

		stage('current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'jenkins') {
					sh '''
						kubectl config use-context arn:aws:eks:us-east-2:549195752255:cluster/capstonecluster
					'''
				}
			}
		}

		stage('blue deployment') {
			steps {
				withAWS(region:'us-east-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('green deployment') {
			steps {
				withAWS(region:'us-east-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Waiting user approval') {
            steps {
                input "Would you like to Redirect traffic to green?"
            }
        }

		stage('Redirect to green') {
			steps {
				withAWS(region:'us-east-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}

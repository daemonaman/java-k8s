pipeline {
        agent any
	stages {
                stage('pull code from git repo') {
                       steps {
                                 git branch: 'main', url: 'https://github.com/daemonaman/java-app.git'
		       }
		}
		stage('build the code') {
                       steps {
                                 sh 'sudo mvn dependency:purge-local-repository'
				 sh 'sudo mvn clean package'
		       }
		}
		stage('building docker image') {
                       steps {
                                 sh 'sudo docker build -t java-app:$BUILD_TAG .'
				 sh 'sudo docker tag java-app:$BUILD_TAG daemonaman/java-app:$BUILD_TAG'
		       }  
		}
		stage('push on docker-hub') {
                       steps {
                                 withCredentials([string(credentialsId: 'docker_hub_id1', variable: 'docker_hub_pass_var')]) {
                                 sh 'sudo docker login -u daemonaman -p ${docker_hub_pass_var}'
				 sh 'sudo docker push daemonaman/java-app:$BUILD_TAG'
                                 }
		       }
		}
		stage('testing the build') {
                        steps {
                                 sh 'sudo docker run -dit --name java-container$BUILD_TAG -p 8090:8080 daemonaman/java-app:$BUILD_TAG'
                       }

		}
		stage ("QAT Testing"){
			steps {
				retry(5) {
					script {
						sh 'sudo curl --silent http://52.66.169.148:8090/java-web-app/ | grep -i -E "(india|sr)"'
					}
				}
			}
		}
		stage ("Approval from QAT"){
			steps {
				script {
					Boolean userInput = input(id: 'Proceed1', message: 'Do you want to Promote this build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                				echo 'userInput: ' + userInput
				}
			}
		}
		stage ("Prod ENV"){
		     agent { label 'k8s-agent' }
		     steps {
		         withCredentials([sshUserPrivateKey(credentialsId: 'k8s_id', keyFileVariable: 'k8s_key', passphraseVariable: 'k8s_pass', usernameVariable: 'k8s_user')]) {
                          sh 'kubectl apply -f deployment.yaml'
		          sh 'kubectl apply -f service.yaml'
			            }
            }
        }
    }
}

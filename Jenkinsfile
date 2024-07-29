pipeline {
        agent any
	stages {
                stage('pull docker image') {
                       steps {
                                 sh 'docker pull daemonaman/java-app:jenkins-java-app-20'
		       }
		}
		stage('testing the build') {
                        steps {
                                 sh 'sudo docker run -dit --name java-container -p 8090:8080 daemonaman/java-app:jenkins-java-app-20'
                       }

		}
		
		stage ("QAT Testing"){
			steps {
				retry(5) {
					script {
						sh 'sudo curl --silent http://13.201.117.124:8090/java-web-app/ | grep -i -E "(india|sr)"'
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

currentBuild.displayName = "Password-Manager # "+currentBuild.number

   def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
        
pipeline {

  agent any
  
  environment{
	Docker_tag = getDockerTag()
    }

  parameters {
	choice(name: 'action', choices: 'create\ndelete', description: 'Create/delete of the deployment')
  }
    
  stages{
	stage("Docker Build and Push") {
	    when {
			expression { params.action == 'create' }
			}
	    steps {
            script {
                sh 'docker build . -t <add your docker hub repo here >:$Docker_tag'
                withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
                    sh 'docker login -u <docker-user-name> -p $docker_password'
                    sh 'docker push <docker-repository>:$Docker_tag'
            }
	    }
    }   
 }
	stage("Create deployment") {
		when {
			expression { params.action == 'create' }
			}
	    steps {
            script {
	            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    credentialsId: 'AWS_4_TERRAFORM', 
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
	                withCredentials([kubeconfigFile(credentialsId: 'kubernetes_config', 
	                    variable: 'KUBECONFIG')]) {
	                    sh 'kubectl create -f password-manager.yaml'
	                    }
	                }
	            }
	        }
	    }

		stage("delete deployment") {
			when {
				expression { params.action == 'delete' }
			}
	        steps {
                script {
	                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        credentialsId: 'AWS_4_TERRAFORM', 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
	                        withCredentials([kubeconfigFile(credentialsId: 'kubernetes_config', 
	                        variable: 'KUBECONFIG')]) {
                              sh 'kubectl create -f password-manager.yaml'
                            }
	               }
	            }
	        }
	    }
    }
}

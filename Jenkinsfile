pipeline {
  agent any
  tools { 
        maven 'Maven_3_8_4'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-jenkins-playground -Dsonar.organization=devsecops-jenkins-playground -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=d6af8c00f3a638235980cd5f7691d40b76178c8c'
			}
        } 
	
	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    	}

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("branci21")
                	}
            	}
        	}
    	}

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://851725524544.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                    app.push("latest")
                	}
            	}
        	}
    	}
	
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
		steps {
			withKubeConfig([credentialsId: 'kubelogin']) {
			sh	('kubectl delete all --all -n devsecops')
			sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
				}
			}
   		}

	stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   		}
		}
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/branci21buggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    	}
	    	}
		} 
  }
}

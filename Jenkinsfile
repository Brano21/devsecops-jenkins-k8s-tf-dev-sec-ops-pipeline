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
  }
}

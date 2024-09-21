pipeline {
  agent any
  tools { 
        maven 'Maven_3_8_4'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=asgbuggywebapplication1_asgbuggywebapplication1 -Dsonar.organization=asgbuggywebapplication1 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=4bd04bcab91d63180c6fa5ab5cc278bb1c17489e'
			}
    }
	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'Synk_token', variable: 'c9f48106-d056-4e6a-ac93-6c4d6703eb3f')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }
	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: "https://us-east-1.console.aws.amazon.com/ecr/repositories/private/730335464014/asgbuggywebapplication1?region=us-east-1"]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://us-east-1.console.aws.amazon.com/ecr/repositories/private/730335464014/asgbuggywebapplication1?region=us-east-1', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")                                                                        
                    }
                }
            }
    	}
	   stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
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
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 



   }
}

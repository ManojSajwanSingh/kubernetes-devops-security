pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //test test.....,...
            }
        }
      
  	  
      stage('Unit Tests - JUnit and Jacoco') {
					steps {
						sh "mvn test"
					}
					post {
						always {
							junit 'target/surefire-reports/*.xml'
							jacoco execPattern: 'target/jacoco.exec'
              
                }
       	    }					
         }
	  
     stage('Mutation Tests - PIT') {
	      steps {
		sh "mvn org.pitest:pitest-maven:mutationCoverage"
	      }
	      post {
		always {
		  pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
		}
	      }
	    }
	  
	
      stage('Sonar Qube - SAST') {
            steps {
		    withSonarQubeEnv('SonarQube') {
		      sh " mvn sonar:sonar \
		  -Dsonar.projectKey=numeric-application \
		  -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000 \
		  -Dsonar.login=0fd6dc8d89f33e9a5ef5d88dd9a5d4c26eef2b92"
			    }
		    timeout(time:2, unit: 'MINUTES') {
			    script {
				    waitforQualityGate abortPipeline: true
			    }
			}
	      }
	  
	  
	  
      stage('Docker Build and Push') {
         steps {
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
		  sh 'printenv'
		  sh 'docker build -t araditsajwan/numeric-app:""$GIT_COMMIT"" .'
		  sh 'docker push araditsajwan/numeric-app:""$GIT_COMMIT""'
                  }
                }
            }
        
	stage('Kubernetes Deployment - DEV') {
	      steps {
		withKubeConfig([credentialsId: 'kubeconfig']) {
		  sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
		  sh "kubectl apply -f k8s_deployment_service.yaml"
		}
           }
      }
  }    
  }


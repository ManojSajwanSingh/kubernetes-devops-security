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
	  
	
      stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000 -Dsonar.login=17fecb7328507c79417552946f6b774e33862b21"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
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



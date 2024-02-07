pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded 
            }
        }   



         stage('UNIT test & jacoco ') {
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
stage('Vulnerability Scan - Docker Trivy') {
       steps {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              withCredentials([string(credentialsId: 'trivy_corentin', variable: 'TOKEN')]) {
                sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh"      
                  sh "sudo bash trivy-image-scan.sh"
              }
              
            }
       }

}
stage('Vulnerability Scan - Docker') {
   steps {
    	catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
			 sh "mvn dependency-check:check"
    	}
   	 }
   	 post {
  	always {
   			 dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
   			 }
   	 }
 }


stage('Docker Build and Push') {
  	steps {
    	withCredentials([string(credentialsId: 'docker-hub-thorondor1', variable: 'DOCKER_HUB_PASSWORD')]) {
      	sh 'sudo docker login -u thorondor1 -p $DOCKER_HUB_PASSWORD' //test
      	sh 'printenv'
      	sh 'sudo docker build -t thorondor1/devops-app:""$GIT_COMMIT"" .'
      	sh 'sudo docker push thorondor1/devops-app:""$GIT_COMMIT""'
    	}

  	}
	}


stage('SonarQube - SAST') {
     	 
       	steps {
   		catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
     	withSonarQubeEnv('SonarQube') {


        	sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=projectcorentin \
  -Dsonar.projectName='projectcorentin' \
  -Dsonar.host.url=http://mytpm.eastus.cloudapp.azure.com:9112 \
  -Dsonar.token=sqp_d183b92e8c9449be24c65c8ac48b02aa0e69212d"
     	}
 
   		}
   	}
     	 
 
 	}

    }
}
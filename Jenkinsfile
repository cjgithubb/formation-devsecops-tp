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
//--------------------------replace variable  token_github on file trivy-image-scan.sh
         withCredentials([string(credentialsId: 'trivy_corentin', variable: 'TOKEN')]) {
              catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              
                sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh"      
                  sh "sudo bash trivy-image-scan.sh"
              }


        
        
        }
       }

}


stage('Docker Build and Push') {
  	steps {
    	withCredentials([string(credentialsId: 'docker-hub-thorondor1', variable: 'DOCKER_HUB_PASSWORD')]) {
      	sh 'sudo docker login -u thorondor1 -p $DOCKER_HUB_PASSWORD'
      	sh 'printenv'
      	sh 'sudo docker build -t thorondor1/devops-app:""$GIT_COMMIT"" .'
      	sh 'sudo docker push thorondor1/devops-app:""$GIT_COMMIT""'
    	}

  	}
	}

    }
}
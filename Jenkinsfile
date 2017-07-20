pipeline {
    options { 
        timeout(time: 5, unit: 'MINUTES')
		buildDiscarder(logRotator(numToKeepStr: '5')) 
        skipDefaultCheckout() 
    }
    agent none

	script {artifactName='*.jar'}
  
  	stages {
		stage('Checkout') {
			agent { label 'docker-cloud' }
				steps {
					echo 'INFO - Retrieving Source'
			//		git 'https://github.com/jglick/simple-maven-project-with-tests.git'
					checkout scm        // create a multi-branch project that only checks out this branch
					gitShortCommit(7)
				}
			}
		stage('Build') {
		  steps {
			echo 'INFO - Starting Build phase'
			sh 'mvn validate'
	//		sh 'mvn compile'
			sh 'mvn clean install' // clean install does a compile, so no reason to do compile, also runs unit tests
		  }
		}
		stage('Test') {
		  steps {
			echo 'INFO - Starting Test phase'
			parallel(
				"unit test": {
					echo 'unit tests'
					mvn test
					mvn surefire-report:report			    // generate a maven unit test report using surefire
					// this is the path to your unit test report: your-project/target/site/surefire-report.html
				},
				"integration tests": {
					echo 'integration tests'
					mvn verify -fn   						// generate a maven integration test report
					junit '**/target/surefire-reports/TEST-*.xml'    // generate a junit test report
			//		archive 'target/*.jar' // i thought we were only going to archive on success, so this should just be a stash and the artifact for the todo-api should be *.war
				}
			)
		  }
		}
		stage('Package') {
		  steps {
			echo 'INFO - Starting Package phase'
			sh 'mvn clean package'
			stash name: 'artifactName', includes: '*.xml'
			}
		}
		stage('Deploy') {
		  steps {
			echo 'INFO - Starting Deploy phase'
	//-------------------------------------------------------------
	//		sh 'mvn deploy' // this won't work until the todo-api pom is not configured to deploy
	//		sh """
	//   		rm -rf ${somevalue}/webapps/ROOT                         // remove a directory and files without comfirmation
	//			mv -f ${somevalue}                               // move files from one directory to another without prompting
	//			cp -rf ${war} ${somevalue}/webapps/ROOT.war      // copy files from one location to another without prompting
	//			${somevalue}/bin/startup.sh
	//		"""
	//-------------------------------------------------------------
				}
			}
		}
	post {				
		success {
			unstash 'artifactName'
			archive "target/${artifactName}"
		}
	}	
}

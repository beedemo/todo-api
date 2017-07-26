pipeline {
    options { 
        timeout(time: 5, unit: 'MINUTES')
		buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
	agent 'maven-jdk-8'
	script { artifactName='*.jar' }
  
  	stages {
		stage('Build') {
			steps {
				echo 'INFO - Starting Build phase'
				sh 'mvn clean install'
		  	}
		}
		stage('Test') {
			steps {
				parallel(
					"unit test": {
						echo 'unit tests'
						sh 'mvn test'
						sh 'mvn surefire-report:report'			// generate a maven unit test report using surefire
						// this is the path to your unit test report: your-project/target/site/surefire-report.html
					},
					"integration tests": {
						echo 'integration tests'
						sh 'mvn verify'    						// generate a maven integration test report
					},	
					"other tests": {
						echo 'other tests'
						junit '**/target/surefire-reports/TEST-*.xml'    // generate a junit test report
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
				sh 'mvn deploy'
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

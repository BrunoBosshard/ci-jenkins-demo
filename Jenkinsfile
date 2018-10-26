node('master') {
	stage('Poll') {
		checkout scm
		env.POMPATH = "${env.WORKSPACE}"
	}
	stage('Build and Unit Test'){
		sh 'mvn clean verify -DskipITs=true';
		junit '**/target/surefire-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
	stage('SonarQube Scan and Quality Gate') {
		node {
			withSonarQubeEnv('Default SonarQube server') {
				sh 'mvn clean verify -f $POMPATH/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
				timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
					def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
					if (qg.status != 'OK') {
						error "Pipeline aborted due to quality gate failure: ${qg.status}"
					}
				}
			} // SonarQube taskId is automatically attached to the pipeline context
		}
	}
	stage ('Integration Test'){
		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
	stage ('Publish'){
		def server = Artifactory.server 'Default Artifactory Server'
		def uploadSpec = """{
			"files": [
				{
					"pattern": "target/hello-0.0.1.war",
					"target": "example-project/${BUILD_NUMBER}/",
					"props": "Integration-Tested=Yes;Performance-Tested=No"
				}
			]
		}"""
		server.upload(uploadSpec)
	}
}

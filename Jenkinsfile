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
	stage('SonarQube Scan') {
			withSonarQubeEnv('Default SonarQube server') {
				sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
				// sh 'mvn clean verify -f $POMPATH/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
			}
	}
	stage('SonarQube Quality Gate') {
		sh 'cat target/sonar/report-task.txt'
		def props = readProperties file: 'target/sonar/report-task.txt'
		def sonarServerUrl = props['serverUrl']
		def ceTaskUrl = props['ceTaskUrl']
		def ceTask
		timeout(time: 1, unit: 'MINUTES') {
			waitUntil {
				def response = httpRequest ceTaskUrl
				ceTask = readJSON text: response.content
				echo ceTask.toString()
				return"SUCCESS".equals(ceTask["task"]["status"])
			}
		}
		def response = httpRequest url : sonarServerUrl + "/api/qualitygates/project_status?analysisId="+ ceTask["task"]["analysisId"], authentication: 'sonarqube-account'
		def qualitygate =  readJSON text: response.content
		echo qualitygate.toString()
		if("ERROR".equals(qualitygate["projectStatus"]["status"])) {
			error  "SonarQube Quality Gate failure"
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

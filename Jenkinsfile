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
		node {
			withSonarQubeEnv('Default SonarQube server') {
				sh 'mvn clean verify -f $POMPATH/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
			} // SonarQube taskId is automatically attached to the pipeline context
		}
	}
	stage('Quality Gate') {
		sh 'sleep 10'
		sh 'cat $POMPATH/target/sonar/report-task.txt'
		defprops = readProperties file: '$POMPATH/target/sonar/report-task.txt'
		defsonarServerUrl=props['serverUrl']
		defceTaskUrl= props['ceTaskUrl']
		defceTask
		timeout(time: 1, unit: 'MINUTES') {
			waitUntil {
				defresponse = httpRequest ceTaskUrl
				ceTask = readJSON text: response.content
				echo ceTask.toString()
				return"SUCCESS".equals(ceTask["task"]["status"])
			}
		}
		defresponse = httpRequest url : sonarServerUrl + "/api/qualitygates/project_status?analysisId="+ ceTask["task"]["analysisId"], authentication: 'jenkins-account'
		defqualitygate =  readJSON text: response.content
		echo qualitygate.toString()
		if("ERROR".equals(qualitygate["projectStatus"]["status"])) {
			error  "Quality Gate failure"
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

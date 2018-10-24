node('master') {
	stage('Poll') {
		checkout scm
	}
	stage('Build and Unit test'){
		sh 'mvn clean verify -DskipITs=true';
		junit '**/target/surefire-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
	stage('SonarQube Scan') {
		node {
			withSonarQubeEnv('Default SonarQube server') {
				echo "${env.CHANGE_ID}"
				echo '$CHANGE_URL'
				echo '$CHANGE_TITLE'
				echo '$CHANGE_AUTHOR'
				echo '$CHANGE_AUTHOR_DISPLAY_NAME'
				echo '$CHANGE_AUTHOR_EMAIL'
				echo '$CHANGE_TARGET'
				echo '$BUILD_NUMBER'
				echo '$BUILD_ID'
				echo '$BUILD_DISPLAY_NAME'
				echo '$JOB_NAME'
				echo '$EXECUTOR_NUMBER'
				echo '$NODE_NAME'
				echo '$NODE_LABELS'
				echo '$WORKSPACE'
				echo '$JENKINS_HOME'
				echo '$JENKINS_URL'
				echo '$BUILD_URL'
				echo '$JOB_URL'
				sh 'mvn clean verify -f $JOB_NAME/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
//				sh 'mvn clean verify -f /var/lib/jenkins/workspace/ci-demo/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
			} // SonarQube taskId is automatically attached to the pipeline context

		}
	}
	stage('Quality Gate') {
		timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
			def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
			if (qg.status != 'OK') {
				error "Pipeline aborted due to quality gate failure: ${qg.status}"
			}
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

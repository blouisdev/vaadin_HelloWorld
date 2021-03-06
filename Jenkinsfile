pipeline {
    agent {
        docker {
            image 'maven:3.8.5-openjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
		stage('SonarQube analysis') {
            steps {
				withSonarQubeEnv('sonar') {
					sh 'mvn sonar:sonar'
				}
			}
		}
		stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage("Deploy") {
            steps {
              archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
            }
        }
        stage ('Upload file') {
            steps {
                rtUpload (
                    // Obtain an Artifactory server instance, defined in Jenkins --> Manage Jenkins --> Configure System:
                    serverId: 'jfrog',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "target/mytodo-1.0-SNAPSHOT.jar",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
            }
        }
    }
}
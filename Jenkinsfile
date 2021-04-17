pipeline {
    agent any
    tools {
        maven 'MAVEN_HOME' 
    }
    stages {
        stage ('Check maven version') {
            steps {
                sh 'mvn --version'
            }
        }
        stage ('Generate package') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Pushing artifacts to Artifactory') {
	    steps {
            rtUpload (
                serverId: 'central',
                spec: '''{
                    "files": [
                        {
                            "pattern": "target/*.jar",
                            "target": "backend/"
                        }
                    ]
                }''',
                buildName: "${env.JOB_NAME}",
                buildNumber: "${env.BUILD_NUMBER}" 
            )
	    }
	}
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "central"
                )
            }
        }
	stage('Sonarqube analysis') {
	    environment { 
	        scannerhome = tool 'scan'
	    }
	    steps {
	        withSonarQubeEnv('sonarqube') {
	        sh "${scannerHome}/bin/sonar-scanner"
	        }
            }
	   }
	stage('Docker Build and Push') {
            steps {
                script {
                    myapp = docker.build("thangaduraisrt/easyclaim-backend:${env.BUILD_ID}")
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_credential') {
                        myapp.push("latest")
                        myapp.push("${env.BUILD_ID}")
                    }
                }
                sh "docker image prune -a -f"
            }
        }

    }
  }

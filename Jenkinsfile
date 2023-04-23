pipeline {
    agent { label  'chef-u16desk'}
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "eliassal/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dh-creds') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        
        stage('ReplaceVariables') {
           steps {
               variableReplace(
					configs: [
						variablesReplaceConfig(
							configs: [
								variablesReplaceItemConfig( 
									name: '$DOCKER_IMAGE_NAME',
									value: '$DOCKER_IMAGE_NAME'
								),
								variablesReplaceItemConfig( 
									name: '$BUILD_NUMBER',
									value: '$BUILD_NUMBER'
								)
							],
							fileEncoding: 'UTF-8', 
							filePath: 'train-schedule-kube.yml', 
							variablesPrefix: '#{', 
							variablesSuffix: '}#'
							)]
				)
            }
	}
        
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh 'kubectl apply -f train-schedule-kube.yml'
                sh 'kubectl rollout status deployment/train-schedule-kube.yml'
            }
        }
    }
}

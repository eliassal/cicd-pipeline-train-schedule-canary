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
	
	stage('ReplaceVariables For Canary Files') {
	   environment { 
                CANARY_REPLICAS = 1
            }
           steps {
              contentReplace(
					configs: [
						variablesReplaceConfig(
							configs: [
								variablesReplaceItemConfig( 
									name: 'DOCKER_IMAGE_NAME',
									value: DOCKER_IMAGE_NAME
								),
								variablesReplaceItemConfig( 
									name: 'BUILD_NUMBER',
									value: '$BUILD_NUMBER'
								),
								variablesReplaceItemConfig( 
									name: 'CANARY_REPLICAS',
									value: CANARY_REPLICAS
							],
							fileEncoding: 'UTF-8', 
							filePath: 'train-schedule-kube-canary.yml', 
							variablesPrefix: '#{', 
							variablesSuffix: '}#'
							)]
				)
            	}
	    }
	    
	stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh 'kubectl apply -f train-schedule-kube-canary.yml'
            }
        }
        
        stage('ReplaceVariables') {
           steps {
              contentReplace(
					configs: [
						variablesReplaceConfig(
							configs: [
								variablesReplaceItemConfig( 
									name: 'DOCKER_IMAGE_NAME',
									value: DOCKER_IMAGE_NAME
								),
								variablesReplaceItemConfig( 
									name: 'BUILD_NUMBER',
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
        
        stage('K8S - DeployToProduction') {
            steps {
		input 'Deploy to Production?'
                milestone(1)
                sh 'kubectl apply -f train-schedule-kube.yml'
                
                
            }
        }
        
        
    }
}

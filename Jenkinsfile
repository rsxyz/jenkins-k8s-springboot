
pipeline {
	agent any

	environment {
		dockerHome = tool 'myDocker'
		mavenHome  = tool 'myMaven'
		PATH = "$dockerHome/bin:$mavenHome/bin:$PATH"
	}

	stages {
		
		stage('Checkout') {
			steps {
				checkout scm
				echo "$PATH"
				echo "$env.BUILD_NUMBER"
				echo "$env.BUILD_ID"
				echo "$env.JOB_NAME"
				echo "$env.BUILD_TAG"
				echo "$env.BUILD_URL"
			}
        }
        stage('Install kubectl'){
            steps{
                sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl && mv kubectl /usr/local/sbin'
            }
        }
        stage('mvn Build'){
            steps {
                sh "mvn clean compile"
            }
        }
        stage('mvn Package'){
            steps {
                sh "mvn package -DskipTests"
                    archiveArtifacts artifacts: 'target/*', fingerprint: true
            }	
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("rsxyz123/hello-springboot:${env.BUILD_ID}")
                }
            }
            
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry( '', 'dockerhub' ) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
            
        }

        stage('Deploy App') {
            steps {
                withKubeConfig(credentialsId: 'KUBECONFIG') {
                    sh "kubectl get nodes"
                    sh "kubectl get pods"
                    sh "cat k8s-hello-springboot.yaml"
                    sh "kubectl  apply -f k8s-hello-springboot.yaml"
                    sh "kubectl get all"
                }
            }
        }
    }
}




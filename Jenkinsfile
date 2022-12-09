
pipeline {
	agent any

    tools{
        maven "MAVEN"
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
                sh 'chmod +x ./kubectl'
            }
        }
        stage('mvn Build'){
            steps {
                sh "java -version"
                sh 'mvn clean compile'
            }
        }
        stage('mvn Package'){
            steps {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: 'target/*', fingerprint: true
            }	
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t rsxyz123/hello-springboot:${env.BUILD_ID} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_PWD', variable: 'dockerPassword')]) {
                    sh 'docker login -u rsxyz123 -p $dockerPassword'
                }
                sh "docker push rsxyz123/hello-springboot:${env.BUILD_ID}"
            } 
        }

        stage('Deploy App') {
            steps {
                withKubeConfig(credentialsId: 'KUBECONFIG') {
                    script{
                        imageName = "rsxyz123/hello-springboot:${env.BUILD_ID}"
                    }
                    sh "kubectl get nodes"
                    sh "kubectl get pods"
                    sh "sed -i 's#replace#${imageName}#g' k8s-hello-springboot.yaml"
                    sh "cat k8s-hello-springboot.yaml"
                    sh "kubectl apply -f k8s-hello-springboot.yaml"
                    sh "kubectl get all"
                }
            }
        }
    }
}



